# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

Laravel 12 REST API プロジェクト（PHP 8.2+）。現在は `feature/todo-crud` ブランチ上のスキャフォールド状態で、ドメインモデルやAPIルートはまだ存在しない。目標はTodo CRUD APIの構築。

## コマンド

```bash
# 開発サーバー起動（サーバー・キュー・Viteを同時起動）
composer dev

# 全テスト実行
composer test

# 特定テストのみ実行
php artisan test --filter="TodoTest"

# コード整形（Laravel Pint）
./vendor/bin/pint

# マイグレーション実行
php artisan migrate

# マイグレーションをリセットしてシードも実行
php artisan migrate:fresh --seed

# APIサポートをインストール（routes/api.php + Sanctum）
php artisan install:api
```

## アーキテクチャ

LaravelのMVC構成。APIプロジェクトとしての役割分担:

- **ルート**: `routes/api.php` — すべてのAPIエンドポイントをここに定義（`/api` プレフィックスは自動付与）
- **コントローラ**: `app/Http/Controllers/` — ビューではなく `response()->json()` または API Resource を返す
- **モデル**: `app/Models/` — `$fillable` を必ず定義してマスアサインメントを防ぐ
- **マイグレーション**: `database/migrations/` — DBスキーマの唯一の定義元
- **テスト**: `tests/Feature/` — Pest使用、`tests/Pest.php` でセットアップ

`routes/web.php` は Blade のウェルカムページを返すだけで、API開発では使わない。

## テスト

[Pest](https://pestphp.com/)（Laravelプラグイン付き）を使用。Feature テストは `Tests\TestCase` を継承（`tests/Pest.php` で設定済み）。`RefreshDatabase` はコメントアウト中 — DBを使うテストを書く時に有効化する。

```php
test('Todo一覧が取得できる', function () {
    $response = $this->getJson('/api/todos');
    $response->assertStatus(200);
});
```

## データベース

- ドライバ: MySQL（データベース名: `todo_api`）
- 現在はフレームワーク標準テーブルのみ（users, cache, jobs）
- キュー・キャッシュ・セッションはすべてデータベース管理
