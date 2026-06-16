# ハッカソン練習計画 — Laravel API × Git チーム開発

## ゴール
- フロントと別リポジトリで動く Todo CRUD API を `todo-api` で完成させる
- Git のブランチ・PR ワークフローを体で覚える

---

## フェーズ一覧

| フェーズ | 内容 | 目安時間 |
|---|---|---|
| 0 | 環境確認・Git 初期設定 | 15分 |
| 1 | API の土台を作る | 30分 |
| 2 | Todo CRUD を実装する | 60分 |
| 3 | 動作確認・テスト | 30分 |
| 4 | PR ワークフローを練習する | 30分 |

---

## フェーズ 0　環境確認・Git 初期設定

```bash
# 現在のブランチを確認
git branch

# feature/todo-crud ブランチにいることを確認（なければ作る）
git switch -c feature/todo-crud

# MySQL が動いているか確認（XAMPP の Apache/MySQL を起動しておく）
php artisan migrate
```

**チェック:**
- [ ] `feature/todo-crud` ブランチにいる
- [ ] `php artisan migrate` がエラーなく通る

---

## フェーズ 1　API の土台を作る

### 1-1. API モードを有効化

```bash
php artisan install:api
```

> `routes/api.php` が生成される。Laravel 12 ではこのコマンドが必要。

**コミット:**
```bash
git add routes/api.php bootstrap/app.php config/sanctum.php
git commit -m "add: APIルートとSanctumを初期化"
```

### 1-2. マイグレーションを作る

```bash
php artisan make:model Todo -m
```

`database/migrations/xxxx_create_todos_table.php` を開いて編集:

```php
public function up(): void
{
    Schema::create('todos', function (Blueprint $table) {
        $table->id();
        $table->string('title');
        $table->boolean('is_done')->default(false);
        $table->timestamps();
    });
}
```

```bash
php artisan migrate
```

**コミット:**
```bash
git add app/Models/Todo.php database/migrations/
git commit -m "add: Todoモデルとマイグレーションを追加"
```

---

## フェーズ 2　Todo CRUD を実装する

### 2-1. 必要なファイルを一括生成

```bash
php artisan make:controller TodoController --api --model=Todo
php artisan make:request StoreTodoRequest
php artisan make:request UpdateTodoRequest
php artisan make:resource TodoResource
```

### 2-2. モデルに `$fillable` を設定

`app/Models/Todo.php`:

```php
protected $fillable = ['title', 'is_done'];
```

### 2-3. バリデーションを書く

`app/Http/Requests/StoreTodoRequest.php`:

```php
public function authorize(): bool { return true; }

public function rules(): array
{
    return [
        'title' => ['required', 'string', 'max:255'],
    ];
}
```

`app/Http/Requests/UpdateTodoRequest.php`:

```php
public function authorize(): bool { return true; }

public function rules(): array
{
    return [
        'title'   => ['sometimes', 'string', 'max:255'],
        'is_done' => ['sometimes', 'boolean'],
    ];
}
```

### 2-4. Resource でレスポンス形式を固定

`app/Http/Resources/TodoResource.php`:

```php
public function toArray(Request $request): array
{
    return [
        'id'         => $this->id,
        'title'      => $this->title,
        'is_done'    => $this->is_done,
        'created_at' => $this->created_at,
    ];
}
```

### 2-5. コントローラを実装

`app/Http/Controllers/TodoController.php`:

```php
use App\Http\Requests\StoreTodoRequest;
use App\Http\Requests\UpdateTodoRequest;
use App\Http\Resources\TodoResource;
use App\Models\Todo;

public function index()
{
    return TodoResource::collection(Todo::all());
}

public function store(StoreTodoRequest $request)
{
    $todo = Todo::create($request->validated());
    return new TodoResource($todo), 201;
}

public function show(Todo $todo)
{
    return new TodoResource($todo);
}

public function update(UpdateTodoRequest $request, Todo $todo)
{
    $todo->update($request->validated());
    return new TodoResource($todo);
}

public function destroy(Todo $todo)
{
    $todo->delete();
    return response()->noContent();
}
```

### 2-6. ルートを登録

`routes/api.php`:

```php
use App\Http\Controllers\TodoController;

Route::apiResource('todos', TodoController::class);
```

**コミット:**
```bash
git add app/ routes/api.php
git commit -m "add: Todo CRUDコントローラとバリデーションを実装"
```

---

## フェーズ 3　動作確認・テスト

### 3-1. サーバーを起動

```bash
php artisan serve
```

### 3-2. Postman / Thunder Client で叩く

| 確認項目 | メソッド | URL | Body |
|---|---|---|---|
| 一覧取得 | GET | `/api/todos` | なし |
| 作成 | POST | `/api/todos` | `{ "title": "牛乳を買う" }` |
| 1件取得 | GET | `/api/todos/1` | なし |
| 更新 | PUT | `/api/todos/1` | `{ "is_done": true }` |
| 削除 | DELETE | `/api/todos/1` | なし |
| バリデーション確認 | POST | `/api/todos` | `{ "title": "" }` → 422が返るか |

### 3-3. Pest でテストを書く（余裕があれば）

`tests/Feature/TodoTest.php` を新規作成:

```php
<?php

use App\Models\Todo;

test('Todo一覧が取得できる', function () {
    Todo::factory()->count(3)->create();
    $this->getJson('/api/todos')->assertStatus(200)->assertJsonCount(3, 'data');
});

test('Todoが作成できる', function () {
    $this->postJson('/api/todos', ['title' => 'テスト'])->assertStatus(201);
});

test('titleが空だと422になる', function () {
    $this->postJson('/api/todos', ['title' => ''])->assertStatus(422);
});
```

```bash
php artisan test
```

**コミット:**
```bash
git add tests/
git commit -m "add: Todo APIのPestテストを追加"
```

---

## フェーズ 4　PR ワークフローを練習する

### 4-1. GitHub にリポジトリを作って push

```bash
# GitHub でリポジトリ作成後
git remote add origin https://github.com/<ユーザー名>/todo-api.git
git push -u origin feature/todo-crud
```

### 4-2. GitHub で PR を作成

1. GitHub を開く
2. 「Compare & pull request」ボタンをクリック
3. タイトル: `add: Todo CRUD API を実装`
4. 説明に **エンドポイント表** を貼る（フロントへの契約）:

```markdown
## エンドポイント

| メソッド | パス | 説明 |
|---|---|---|
| GET | /api/todos | 一覧取得 |
| POST | /api/todos | 作成 |
| GET | /api/todos/{id} | 1件取得 |
| PUT | /api/todos/{id} | 更新 |
| DELETE | /api/todos/{id} | 削除 |

## レスポンス例（作成）
POST /api/todos
Body: { "title": "牛乳を買う" }
Response 201:
{ "data": { "id": 1, "title": "牛乳を買う", "is_done": false, "created_at": "..." } }
```

5. 「Create pull request」→ 自分でレビュー → `main` にマージ

### 4-3. マージ後のローカル整理

```bash
git switch main
git pull origin main          # マージされた内容を取り込む
git branch -d feature/todo-crud  # 終わったブランチを削除
```

---

## ハッカソン当日のチェックリスト

- [ ] 朝イチで `git pull origin main`
- [ ] 作業前に `git switch -c feature/<機能名>` でブランチを切る
- [ ] こまめにコミット（30分に1回目安）
- [ ] 機能が完成したら PR を作ってチームに共有
- [ ] `.env` は絶対にコミットしない
- [ ] フロントに変更が生じたら口頭 + チャットで共有する

---

## CORS 設定（フロントから叩かせる時）

`config/cors.php`:

```php
'allowed_origins' => ['http://localhost:3000', '*'],  // 開発中は * でOK
```
