# Cornix LED インジケーター仕様メモ

このメモは、このリポジトリの現行設定を前提にした整理です。

- `build.yaml` では `cornix_left` / `cornix_right` に `cornix_indicator` が付いている
- 実装は `zmk-rgbled-widget` を利用している
- 細かなアニメーションや優先順位は upstream 実装依存なので、この文書では「今の設定で何が見えるか」を中心に書く

## 前提

- 各 half に RGB LED は 2 個
- `cornix_indicator` は WS2812 モードで動く
- 明るさは `CONFIG_RGBLED_WIDGET_BRIGHTNESS=64`
- `CONFIG_RGBLED_WIDGET_LED_SHARING=y` は有効
- ただし、レイヤー表示そのものは今の設定では無効

## LED の役割

| LED | 左側（central） | 右側（peripheral） |
|------|------------------|--------------------|
| LED 0 | 🔋 バッテリー残量 | 🔋 バッテリー残量 |
| LED 1 | 📶 ホストとの接続状態 | 🔗 左右 split の接続状態 |

補足:

- 右側はホスト PC との BLE プロファイル状態は表示しない
- Bluetooth プロファイル切替の反映は左側だけで起こる
- レイヤー情報は central 側しか持たないが、今の設定では LED に出していない

## 起動直後の表示

起動時は、だいたい次の順で状態を出します。

1. バッテリー状態
2. 接続状態

### バッテリー状態

- 🟢 緑: 80% 以上
- 🟡 黄: 20% 以上 80% 未満
- 🔴 赤: 20% 未満
- 🔴 かなり低いときは、赤でより強い警告表示になる

### 接続状態

左側（central）:

- 🔵 青: BLE 接続済み
- 🟡 黄: アドバタイズ中
- 🔴 赤: 未接続

右側（peripheral）:

- 🔵 青: 左側と split 接続済み
- 🔴 赤: 左側と未接続

## 通常時の見え方

- LED 0 は 🔋 バッテリー状態を保持する
- LED 1 は 📶 / 🔗 接続状態を保持する
- 左側で `BT_SEL 0/1/2` を押すと、LED 1 は選択中プロファイルの状態に更新される

## 今の設定で出ないもの

### レイヤー表示

レイヤー表示はモジュール側に機能がありますが、このリポジトリでは未有効です。

- `CONFIG_RGBLED_WIDGET_SHOW_LAYER_CHANGE` は未設定
- `CONFIG_RGBLED_WIDGET_SHOW_LAYER_COLORS` も未設定

そのため、今は

- 🔵 レイヤー番号を点滅回数で出す
- 🎨 レイヤーごとに色を固定表示する
- 🔄 2 個の LED を一時的に借りてレイヤーを出す

といった表示は行いません。

`LED_SHARING=y` は「レイヤー表示を有効にしたときに共有利用できる」状態、という理解でよいです。

### USB 優先の色分け

キーマップには `&out OUT_USB` / `&out OUT_BLE` / `&out OUT_TOG` がありますが、今の設定では 🟦 シアンの USB 優先表示は出ません。

これは `CONFIG_RGBLED_WIDGET_CONN_SHOW_USB` を有効にしていないためです。

つまり、出力先の切替自体はできますが、LED だけでは USB 優先かどうかは判別しづらいです。

## 関連ファイル

- `build.yaml`
- `boards/shields/cornix_indicator/cornix_indicator.conf`
- `boards/shields/cornix_indicator/cornix_indicator.overlay`
- `config/cornix.keymap`
