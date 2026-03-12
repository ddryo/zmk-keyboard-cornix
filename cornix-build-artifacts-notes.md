# Cornix ビルド成果物メモ

更新日: 2026-03-13

## 目的

`build.yaml` から生成される UF2 ファイルの種類が多く見える理由と、実際にどれを使えばよいかを整理する。

## 結論

- 成果物が多いのは、1 つのリポジトリで複数の運用パターンをまとめてビルドしているため。
- `nosd` は別機能ではなく、主に SoftDevice を前提にしないフラッシュ配置の違い。
- 普段使いで必要なのは一部だけで、用途に合わないものは無視してよい。

## `dongle` とは何か

- `dongle` は Cornix 本体とは別に使う、小型の受信機用ボードを指す
- 典型的には `nice_nano_v2` や `seeeduino_xiao_ble` のような別ボードを PC に USB 接続して使う
- 接続イメージは `左右のキーボード -> dongle -> PC` で、PC とは dongle がつながる
- このリポジトリの `dongle` 系成果物は、そうした別ボードを追加で使う人向けのもの

## 今の使い方で不要なもの

- Cornix の左右だけで使っていて、別の受信機ボードを用意していないなら `dongle` 系成果物は不要
- つまり `cornix_dongle_nosd`、`cornix_dongle_screen_xiao`、`cornix_dongle_screen_nicenano` は無視してよい
- `cornix_left_for_dongle_nosd` も dongle 前提の左半分用なので、通常構成では不要
- 通常運用なら見るべきなのは `cornix_left_default_nosd`、`cornix_right_nosd`、必要時の `reset_nosd` で足りる

## 成果物が増える理由

`build.yaml` に複数ターゲットが並んでいるため、用途別の UF2 がまとめて生成される。

主に次の 3 軸が重なっている。

### 1. どのハード向けかが違う

- `cornix_left_default_nosd`
  - 通常の Cornix 左半分用
  - 左側が central になる、いわゆる「dongle なし」構成
- `cornix_left_for_dongle_nosd`
  - dongle 運用時の左半分用
  - `cornix_ph_left` なので、通常 left とは役割が違う
- `cornix_right_nosd`
  - 右半分用
- `cornix_dongle_nosd`
  - 外付け dongle 用
- `cornix_dongle_screen_xiao`
  - Seeed XIAO BLE ベースの画面付き dongle 用
- `cornix_dongle_screen_nicenano`
  - nice!nano v2 ベースの画面付き dongle 用
- `reset` / `reset_nosd`
  - 通常利用ではなく、設定初期化用

### 2. `nosd` は機能差ではなく、フラッシュ配置差

- このリポジトリの README では、Cornix 純正 RMK 由来の事情で SoftDevice なしのレイアウトを扱っている。
- `nosd` は「SoftDevice 領域を前提にしないビルド」という意味で、別機能ではない。
- README にも「このプロジェクトでは no-SD layout をデフォルトにしている」とある。
- そのため、普段使いで必要なのは多くの場合 `*_nosd.uf2` 側になる。

補足:

- `cornix_left_default_nosd` や `cornix_right_nosd` は、board 定義側がすでに no-SD 前提のため、artifact 名に `nosd` が付いていても追加 snippet は使っていない
- `cornix_dongle_nosd` と `reset_nosd` は、`nice_nano_v2` や generic reset build に対して `nrf52840-nosd` snippet を足して no-SD 化している

### 3. README と build.yaml に少し世代差がある

- README の説明では `cornix_left` / `cornix_right` / `cornix_dongle` のような旧来の名前が出てくる
- 一方、現在の `build.yaml` では実際の artifact 名が `*_nosd` ベースになっている
- つまり、「古い説明」と「今の成果物名」が少しずれている

## 実際にどれを使えばよいか

### dongle を使わない場合

- 左: `cornix_left_default_nosd`
- 右: `cornix_right_nosd`
- 必要なときだけ: `reset_nosd`

### dongle を使う場合

- dongle: `cornix_dongle_nosd`
- 左: `cornix_left_for_dongle_nosd`
- 右: `cornix_right_nosd`
- 必要なときだけ: `reset_nosd`

### `reset` / `reset_nosd` について

- これは常用ファームではなく、設定初期化用
- 以前のペアリング情報や保存設定が悪さをしているときに一度焼いてから、本命 UF2 を入れ直すためのもの
- 今の構成を基準にするなら、基本は `reset_nosd` を見ればよい

## 実務上の理解

- 「成果物が多い」のではなく、「複数運用パターンを 1 つのリポジトリでまとめてビルドしている」と考えるのが正しい
- 実際に普段使いで必要なものは、ごく一部だけ
- Cornix 本体向け運用では `nosd` 系だけ見ればほぼ足りる可能性が高い
- `reset` 系と画面付き custom dongle 系は別用途なので、不要なら無視してよい

## 関連ファイル

- [build.yaml](/Users/ryo/DEV/Library/Fork/zmk-keyboard-cornix/build.yaml)
- [README.md](/Users/ryo/DEV/Library/Fork/zmk-keyboard-cornix/README.md)
- [bootloader/README.md](/Users/ryo/DEV/Library/Fork/zmk-keyboard-cornix/bootloader/README.md)
