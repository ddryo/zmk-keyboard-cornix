# Cornix + ZMK 調査メモ

更新日: 2026-03-13

## 目的

Cornix をこのリポジトリの ZMK ファームウェアで使ったときに感じる以下の問題について、設定と実装の観点で整理する。

- 入力遅延が起こる
- 充電状態や電池残量が見えない
- Bluetooth の接続先切替や接続状態のフィードバックがない
- 純正ファームウェアでは光っていたような LED 表示がない

## 結論

- 体感遅延は 1 つの原因ではなく、`split BLE` 固有の遅延と、現在の `hold-tap` 多用キーマップの遅延が重なっている可能性が高い。
- バッテリー情報は「取れていない」というより「見える場所がほぼ無い」状態。
- LED フィードバックは実装の土台はあるが、デフォルトビルドでは無効。
- USB 給電中に BLE で使いたいケースに対して、`&out OUT_BLE` などの出力切替キーが未設定なので、挙動がわかりにくくなっている可能性がある。

## 今回確認できたこと

### 1. Split BLE 由来の遅延は避けられない

- ZMK 公式ドキュメントでは、BLE split の peripheral 側入力は平均 `3.75ms`、最悪 `7.5ms` の追加遅延があるとされている。
- Cornix は split 構成で、左半分または dongle が central、右半分が peripheral になるため、この遅延は構造的に発生しうる。
- つまり、ZMK にした時点で「完全に純正と同じ無遅延」にはならない。

関連:

- 公式: <https://zmk.dev/docs/features/split-keyboards>
- ローカル設定: [boards/arm/cornix/Kconfig.defconfig](boards/arm/cornix/Kconfig.defconfig)

### 2. 今のキーマップは、遅延を感じやすい設定になっている

- ホームロウ Mod が多く、`tap-preferred` + `tapping-term-ms = 180` + `hold-trigger-on-release` の組み合わせになっている。
- これは誤爆防止には有効だが、打鍵時の判定待ちが発生しやすい。
- とくにホーム段や親指キーで「反応が遅い」と感じるなら、BLE より先にこの設定の影響を疑うべき。
- `require-prior-idle-ms` は入っているので高速入力時の遅延を少し抑えているが、全体としてはかなり慎重寄りの設定。

関連:

- キーマップ: [config/cornix.keymap](config/cornix.keymap)
- 特に該当: `hm_l`, `hm_r`, `hm_shift_l`, `hm_shift_r`
- 公式: <https://zmk.dev/docs/keymaps/behaviors/mod-tap>
- 公式: <https://zmk.dev/docs/config/behaviors>

### 3. 電池残量の取得は有効だが、表示手段が弱い

- 左側 central では以下が有効になっている。
  - `CONFIG_ZMK_BATTERY_REPORT_INTERVAL=60`
  - `CONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_PROXY=y`
  - `CONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_FETCHING=y`
- つまり、左右の電池取得をする前提の設定にはなっている。
- ただし、キーボード本体側では `CONFIG_ZMK_DISPLAY=n` なので、本体だけでは見える情報がほぼ無い。
- ZMK 公式でも、split の両側バッテリーをホスト上で正しく見せる仕組みは曖昧で、多くの場合は main battery しか見えないと説明されている。

関連:

- 左 central 設定: [boards/arm/cornix/cornix_left_defconfig](boards/arm/cornix/cornix_left_defconfig)
- 電池センサー定義: [boards/arm/cornix/nrf_e73.dtsi](boards/arm/cornix/nrf_e73.dtsi)
- 公式: <https://zmk.dev/docs/config/battery>
- 公式: <https://zmk.dev/docs/features/battery>

### 4. LED フィードバックはあるが、デフォルトでは無効

- `cornix_indicator` shield に WS2812 と RGB widget の設定がある。
- ただし、デフォルトの `build.yaml` では `shield: cornix_indicator` がコメントアウトされている。
- README にも「RGB led light を有効にできるが、消費電力が増える」と書かれている。
- そのため、純正ファームウェアで見えていたような接続・電池の光フィードバックは、現状の標準ビルドでは出ない。

関連:

- ビルド定義: [build.yaml](build.yaml)
- indicator 設定: [boards/shields/cornix_indicator/cornix_indicator.conf](boards/shields/cornix_indicator/cornix_indicator.conf)
- indicator overlay: [boards/shields/cornix_indicator/cornix_indicator.overlay](boards/shields/cornix_indicator/cornix_indicator.overlay)
- README: [README.md](README.md)

### 5. Bluetooth プロファイル切替はあるが、出力先切替がない

- キーマップには `&bt BT_SEL 0/1/2` や `&bt BT_CLR_ALL` がある。
- つまり、Bluetooth プロファイル切替機能そのものは入っている。
- 一方で、`&out OUT_USB` / `&out OUT_BLE` / `&out OUT_TOG` は割り当てられていない。
- ZMK 公式では、USB と BLE の両方がつながっている場合、既定では USB が優先される。
- そのため、充電中や USB 接続中に「BLE で打っているつもりなのに反応しない」「接続先がわかりにくい」と感じる原因になりうる。

関連:

- BT 切替キー: [config/cornix.keymap](config/cornix.keymap)
- 公式: <https://zmk.dev/docs/keymaps/behaviors/bluetooth>
- 公式: <https://zmk.dev/docs/keymaps/behaviors/outputs>

### 6. Dongle を使う前提の改善ルートは用意されている

- dongle 側には display を使う構成がある。
- `CONFIG_ZMK_DISPLAY=y` で、`cornix_dongle_eyelash` には OLED 定義がある。
- dongle を central にすると、ホストとの接続や表示を本体から切り離しやすく、運用上の見通しは改善しやすい。

関連:

- dongle conf: [boards/shields/cornix_dongle_adapter/cornix_dongle_adapter.conf](boards/shields/cornix_dongle_adapter/cornix_dongle_adapter.conf)
- OLED overlay: [boards/shields/cornix_dongle_eyelash/cornix_dongle_eyelash.overlay](boards/shields/cornix_dongle_eyelash/cornix_dongle_eyelash.overlay)
- ビルド定義: [build.yaml](build.yaml)

### 7. ZMK 本体は古めのバージョンに固定されている

- `config/west.yml` では ZMK を `v0.3` にピン留めしている。
- コメント上も、`main` へ上げるには Zephyr 4.1 / HWMv2 への移行が必要と書かれている。
- そのため、上流の改善をそのまま取り込めていない可能性はある。
- ただし、アップデートは簡単な差し替えでは済まず、別作業として扱うべき。

関連:

- manifest: [config/west.yml](config/west.yml)

## 「充電が少ないとダメ / しすぎてもダメっぽい」について

このリポジトリだけからは、充電量と遅延の因果関係までは断定できない。

ただし、次の 2 点は十分ありうる。

- バッテリー残量低下により無線品質や安定性が悪化している
- USB 給電時に出力先が USB 優先になり、BLE へ出ていないことを「不調」と感じている

少なくとも後者は、`&out` が未割り当てである以上、実運用上の混乱要因としてかなり可能性が高い。

## 改善の優先順位

### 優先度高

- キーマップに `&out OUT_BLE` / `&out OUT_USB` / `&out OUT_TOG` を追加する
- Bluetooth レイヤの配置を見直し、接続先切替と出力先切替を区別して使えるようにする
- ホームロウ Mod の `tapping-term-ms` を短くするか、一部を通常キーに戻して体感差を確認する

### 優先度中

- `cornix_indicator` を有効にして、接続状態や電池状態の LED フィードバックを戻す
- dongle + display 構成に寄せて、状態表示を OLED 側へ出す

### 優先度低

- ZMK 本体のバージョン更新を検討する
- ただし HWMv2 移行コストがあるので、まずはキーマップと表示改善を先にやる方が効果が大きい

## 先にやるとよさそうな実装候補

1. `func_layer` に `&out OUT_BLE` と `&out OUT_USB` を追加する
2. `BT_SEL` 系を整理して、今どのプロファイルを使うかを把握しやすくする
3. `cornix_indicator` を有効化したビルドを 1 つ追加する
4. ホームロウ Mod の `tapping-term-ms` を `180 -> 140` 前後で試す
5. 必要なら dongle 側ビルドを常用前提に寄せる

## 参照

- ZMK Split Keyboards: <https://zmk.dev/docs/features/split-keyboards>
- ZMK Battery Level: <https://zmk.dev/docs/features/battery>
- ZMK Battery Config: <https://zmk.dev/docs/config/battery>
- ZMK Bluetooth Behavior: <https://zmk.dev/docs/keymaps/behaviors/bluetooth>
- ZMK Output Selection Behavior: <https://zmk.dev/docs/keymaps/behaviors/outputs>
- ZMK Hold-Tap Behavior: <https://zmk.dev/docs/keymaps/behaviors/mod-tap>
- ZMK Behavior Config: <https://zmk.dev/docs/config/behaviors>
