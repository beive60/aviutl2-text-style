# aviutl2-text-style

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

AviUtl2 のテキストオブジェクトに文字単位のスタイル調整を適用する Lua アニメーション効果集です。タイトルテロップなどの文字組みに活用できます。

提供するフィルタ：

- **manual_kerning** — 文字間のカーニング値を 1 行ずつ指定（横書き・縦書き対応）
- **manual_font_size** — 文字ごとのフォントサイズを % で指定（下辺揃え対応）

## 必要環境

- AviUtl2（最新版推奨）
- Windows x86_64

## インストール

[Releases](https://github.com/beive60/aviutl2-manual-kerning/releases) から
最新の `@manual_text_style.anm2` をダウンロードします。

1. `@manual_text_style.anm2` を AviUtl2 の Script フォルダにコピーする
   - 既定パス: `C:\ProgramData\AviUtl2\Script\`
2. AviUtl2 を再起動する

## 使い方

### manual_kerning

1. テキストオブジェクトにテキストを入力する
   例: `AviUtl`
2. テキストオブジェクトの「個別オブジェクト」を有効にする
3. アニメーション効果「manual_kerning」または「manual_font_size」をそのテキストオブジェクトに追加する
4. 縦書きテキストの場合は「縦書き」チェックボックスを有効にする
5. 効果の「カーニング値」欄に、文字間ごとのカーニング値を **1 行ずつ** 入力する

例（`AviUtl` 6 文字の場合）:

```
-10
0
5
0
0
```

| 行 | 適用箇所 | 値 |
|---|---|---|
| 1行目 | A と v の間 | -10px |
| 2行目 | v と i の間 | 0px |
| 3行目 | i と U の間 | +5px |
| 4行目 | U と t の間 | 0px |
| 5行目 | t と l の間 | 0px |

空行・数値以外の行は 0 として扱います。

### manual_font_size

1. テキストオブジェクトの「個別オブジェクト」を有効にする
2. アニメーション効果「manual_font_size」をテキストオブジェクトに追加する
3. 効果の「フォントサイズ」欄に、文字ごとのサイズを **% 単位で 1 行ずつ** 入力する

例（`AviUtl` 6 文字の場合）:

```
100
150
80
100
100
100
```

| 行 | 文字 | サイズ |
|---|---|---|
| 1行目 | A | 100%（等倍） |
| 2行目 | v | 150% |
| 3行目 | i | 80% |
| 4行目 | U | 100%（等倍） |
| 5行目 | t | 100%（等倍） |
| 6行目 | l | 100%（等倍） |

空行・数値以外の行は 100%（等倍）として扱います。サイズ変更した文字の下辺は、基準サイズの文字の下辺に揃えられます。

## カーニング値の計算方法

文字 i（0 始まり）の X オフセット = 1行目〜i行目の合計

| 文字 | インデックス | オフセット |
|---|---|---|
| A | 0 | 0 |
| v | 1 | -10 |
| i | 2 | -10 + 0 = -10 |
| U | 3 | -10 + 0 + 5 = -5 |
| t | 4 | -5 + 0 = -5 |
| l | 5 | -5 + 0 = -5 |

「文字間隔」スライダーを使うと、すべての文字間に均等な間隔を加算できます。

## 制限事項

- テキストオブジェクトの「個別オブジェクト」が無効の場合、効果は適用されません。
- 縦書きテキストに使用する際は、効果パネルの「縦書き」チェックボックスを手動で有効にする必要があります。

## プロジェクト構成

```
aviutl2-manual-kerning/
├── scripts/
│   └── @manual_text_style.anm2  # Lua アニメーション効果（本体）
├── aviutl2.toml                 # au2 CLI 設定
└── docs/
    └── architecture.md
```

## 開発

[aviutl2-cli](https://github.com/sevenc-nanashi/aviutl2-cli)（`au2` コマンド）を使用します。

### 初回セットアップ

```powershell
au2 prepare
```

AviUtl2 本体のダウンロードと開発環境のセットアップを行います。

### 開発用 AviUtl2 の起動

```powershell
au2 develop
```

`aviutl2.toml` に `placement_method = "copy"` を設定しているため、シンボリックリンクや開発者モードは不要です。スクリプトをコピーして AviUtl2 を起動します。

## クレジット

- [aviutl2-cli](https://github.com/sevenc-nanashi/aviutl2-cli) — AviUtl2 のスクリプト開発用コマンドラインツール

## コントリビューション

Issue や Pull Request を歓迎します。[GitHub Issues](https://github.com/beive60/aviutl2-manual-kerning/issues) で質問・提案を受け付けています。

詳細は [CONTRIBUTING.md](CONTRIBUTING.md) を参照してください。

## ライセンス

[MIT](LICENSE) © 2026 べいぶ

