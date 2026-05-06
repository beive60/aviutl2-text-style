# Contributing Guide

バグ報告、機能提案、プルリクエストを歓迎します。

## 開発環境のセットアップ

ビルドには以下が必要です。

- [Rust toolchain](https://rustup.rs/) — `x86_64-pc-windows-msvc` ターゲット
- [aviutl2-cli](https://github.com/sevenc-nanashi/aviutl2-cli)（`au2` コマンド）

```powershell
git clone https://github.com/beive60/aviutl2-manual-kerning.git
cd aviutl2-manual-kerning
au2 prepare
au2 develop
cargo test
```

`au2 prepare` は AviUtl2 本体のダウンロードと初期設定を一括で行います。
`au2 develop` はプラグイン（`.aux2`）をビルドして開発用 AviUtl2 ディレクトリに配置します。

Rust 1.85 以降（edition 2024）が必要です。
`rustup update stable` で最新版に更新してください。

## ブランチ戦略

| ブランチ | 用途 |
| --- | --- |
| `main` | 安定リリース |
| `feat/*` | 新機能の開発 |
| `fix/*` | バグ修正 |

## プルリクエストの手順

1. リポジトリをフォークする
2. 作業用ブランチを切る（例: `feat/add-vertical-text-support`）
3. 変更を加えてコミットする
4. `cargo test` と `cargo clippy` がすべて通ることを確認する
5. プルリクエストを `main` ブランチに向けて作成する

## コーディング規約

### フォーマット

`cargo fmt` を実行してからコミットしてください。

```powershell
cargo fmt
cargo clippy -- -D warnings
cargo test
```

### コミットメッセージ

[Conventional Commits](https://www.conventionalcommits.org/) 形式を推奨します。

```txt
feat: プレビューウィンドウのダブルクリックでカーニングをリセット
fix: 日本語テキストの文字境界が 1 文字ずれる問題を修正
docs: アーキテクチャ図に Lua レンダラーフローを追記
```

## バグ報告

[Issues](https://github.com/beive60/aviutl2-manual-kerning/issues) から報告してください。
以下の情報を含めると解決が早くなります。

- AviUtl2 のバージョン
- OS のバージョン（Windows バージョン）
- 再現手順
- 期待する動作と実際の動作
- `RUST_LOG=debug` 時のログ出力（あれば）

## ライセンス

コントリビューションは MIT License に同意したものとみなします。
