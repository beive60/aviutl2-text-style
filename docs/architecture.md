# AviUtl2 Manual Kerning Plugin - Architecture

## 概要

このプラグインは AviUtl2 のテキストオブジェクトのカーニング（文字間隔）を
マウスドラッグ操作で直感的に調整するための FilterPlugin + Lua アニメーション効果の組み合わせシステムです。

---

## システム構成

### コンポーネント一覧

| コンポーネント | ファイル | 言語 | 役割 |
| --- | --- | --- | --- |
| FilterPlugin | `src/lib.rs` | Rust | AviUtl2 プラグインのエントリポイント |
| WndProc フック | `src/plugin/wnd_hook.rs` | Rust | プレビューウィンドウのマウスイベント横取り |
| ヒットテスト | `src/plugin/hit_test.rs` | Rust | マウス座標 → 文字間インデックス変換 |
| タグ編集 | `src/plugin/tag_editor.rs` | Rust | `<k=XX>` タグの挿入・更新・削除 |
| SJIS ユーティリティ | `src/shared/sjis.rs` | Rust | Shift-JIS ↔ UTF-8 変換 |
| Lua レンダラー | `scripts/manual_kerning.anm` | Lua | `<k=XX>` タグを解釈してカーニングを適用 |

---

## データフロー図

### 1. カーニング調整フロー（ドラッグ操作）

```mermaid
sequenceDiagram
    actor User
    participant Preview as プレビューウィンドウ
    participant WndHook as wnd_hook.rs<br/>(WndProc サブクラス)
    participant HitTest as hit_test.rs<br/>(CharBoundaries)
    participant TagEditor as tag_editor.rs
    participant GlobalState as GLOBAL_STATE<br/>(Mutex<PluginState>)

    User->>Preview: 左ボタン押下 (WM_LBUTTONDOWN)
    Preview->>WndHook: kerning_wnd_proc() 呼び出し
    WndHook->>HitTest: hit_test(x, y)
    HitTest-->>WndHook: gap_index: usize
    WndHook->>GlobalState: DragState { gap_index, start_x, ... } を書き込み

    User->>Preview: マウス移動 (WM_MOUSEMOVE)
    Preview->>WndHook: kerning_wnd_proc() 呼び出し
    WndHook->>WndHook: delta = x - last_x
    WndHook->>TagEditor: update_kerning_tag(text, gap_index, delta)
    TagEditor-->>WndHook: 更新後テキスト "A<k=-10>viUtl"
    WndHook->>GlobalState: current_text を更新

    User->>Preview: 左ボタン解放 (WM_LBUTTONUP)
    Preview->>WndHook: kerning_wnd_proc() 呼び出し
    WndHook->>GlobalState: DragState を None にリセット
```

### 2. プラグイン初期化フロー

```mermaid
sequenceDiagram
    participant AviUtl2
    participant Plugin as ManualKerningPlugin<br/>(lib.rs)
    participant WndHook as wnd_hook.rs

    AviUtl2->>Plugin: FilterPlugin::new()
    Plugin->>Plugin: init_logging()
    Plugin->>Plugin: GLOBAL_STATE 初期化
    Plugin-->>AviUtl2: Ok(ManualKerningPlugin)

    AviUtl2->>Plugin: proc_video() (初回フレーム)
    Plugin->>WndHook: ensure_subclassed_with_state() [OnceLock]
    WndHook->>WndHook: find_preview_window()<br/>(EnumChildWindows で最大子ウィンドウを探索)
    WndHook->>WndHook: SetWindowLongPtrW(hwnd, GWLP_WNDPROC, kerning_wnd_proc)
    WndHook->>Plugin: GLOBAL_STATE.original_wnd_proc = 元の WndProc
```

### 3. Lua レンダラーによる描画フロー

```mermaid
flowchart TD
    A[テキストオブジェクト<br/>'A&lt;k=-10&gt;viUtl'] --> B[parse_tokens: テキストを<br/>Char/Kern トークンに分割]
    B --> C{トークン種別}
    C -->|Char| D[obj.copybuffer で<br/>1文字描画バッファを作成]
    C -->|Kern| E[offset_x に kern 値を加算]
    D --> F[obj.draw で offset_x を<br/>適用して描画]
    F --> G[offset_x += 文字幅]
    G --> C
```

---

## モジュール依存関係

```mermaid
graph TD
    lib["lib.rs<br/>(ManualKerningPlugin)"]
    mod_plugin["plugin/mod.rs<br/>(GLOBAL_STATE)"]
    wnd_hook["plugin/wnd_hook.rs<br/>(ensure_subclassed_with_state)"]
    hit_test["plugin/hit_test.rs<br/>(CharBoundaries)"]
    tag_editor["plugin/tag_editor.rs<br/>(update_kerning_tag)"]
    shared_sjis["shared/sjis.rs<br/>(sjis_to_utf8 etc.)"]

    lib --> mod_plugin
    lib --> wnd_hook
    mod_plugin --> hit_test
    wnd_hook --> mod_plugin
    wnd_hook --> tag_editor
    wnd_hook --> hit_test
    tag_editor --> shared_sjis
```

---

## グローバル状態設計

`OnceLock<Mutex<PluginState>>` によるスレッドセーフなグローバル状態管理：

```
GLOBAL_STATE: OnceLock<Mutex<PluginState>>
                 │
                 └── PluginState
                       ├── drag: Option<DragState>         // ドラッグ中の状態
                       │     ├── start_x: i32
                       │     ├── start_y: i32
                       │     ├── gap_index: usize
                       │     └── last_x: i32
                       ├── current_text: String            // 現在のテキスト (UTF-8)
                       ├── boundaries: Option<CharBoundaries>  // ヒットテスト用
                       │     ├── x_positions: Vec<f32>
                       │     ├── y: f32
                       │     └── height: f32
                       └── original_wnd_proc: Option<isize>   // 元の WndProc ポインタ
```

---

## `<k=XX>` タグフォーマット

```
通常テキスト: "AviUtl"
カーニング適用後: "A<k=-10>v<k=5>iUtl"
                    │          │
                    │          └── v と i の間を +5px 広げる
                    └── A と v の間を -10px 縮める
```

- `XX` は符号付き整数（正値 = 広げる、負値 = 縮める）
- 値 0 の場合はタグを自動削除
- 連続する同一位置のタグは合算して正規化

---

## セキュリティ・安全性の考慮事項

1. **HWND を `isize` として保持**: `HWND` は `!Send` のため、グローバル状態には `isize` として保存し、使用時に `HWND(ptr as *mut _)` へキャスト
2. **OnceLock による初期化保証**: WndProc サブクラス化は `OnceLock` で一度だけ実行
3. **Mutex によるドラッグ状態の保護**: `GLOBAL_STATE` は `Mutex` で保護し、複数コールバックからの同時アクセスを防止
4. **Drop での WndProc 復元**: プラグインアンロード時に必ず元の WndProc を復元

---

## インストール手順

1. `cargo build --release` でビルド
2. `target/release/aviutl2_manual_kerning.dll` を AviUtl2 の `Plugin/` フォルダにコピー（`.auf` にリネーム）
3. `scripts/manual_kerning.anm` を AviUtl2 の `script/` フォルダにコピー
4. AviUtl2 を再起動してフィルタ「カーニング」を有効化
5. アニメーション効果「Manual Kerning」をテキストオブジェクトに適用

## 使用方法

1. ExEdit のテキストオブジェクトを選択し、アニメーション効果「Manual Kerning」を追加
2. フィルタ「カーニング」を有効化（プレビューウィンドウの WndProc がサブクラス化される）
3. プレビューウィンドウ上で文字間を左ドラッグして間隔を調整
4. ドラッグ方向: 右ドラッグ = 間隔を広げる、左ドラッグ = 間隔を縮める
