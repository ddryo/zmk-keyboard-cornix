# Cornix HWMv2 移行メモ

## 目的

Zephyr 4 系で導入された Hardware Model v2 (HWMv2) に合わせて、Cornix のボード定義を旧構造から新構造へ移行する際の重点ポイントを整理する。

- 旧構造: `boards/arm/cornix/`
- 新構造: `boards/jzf/cornix/`

## 前提

- このフォークは現時点で `Zephyr 3.5 / ZMK v0.3` のままで、HWMv2 前提ではない。
- HWMv2 は単なるディレクトリ移動ではなく、Zephyr 4 系の board model 前提の変更を含む。
- そのため、`HWMv2 だけ先に入れる` のではなく、`Zephyr 4 系への追従ブランチでまとめて対応する` 方が安全。

## 最重要ポイント

### 1. ディレクトリ移動だけでは終わらない

HWMv2 では、ボード配置が CPU アーキテクチャ基準から vendor 基準に変わる。

- 旧: `boards/<arch>/<board>/`
- 新: `boards/<vendor>/<board>/`

Cornix では `boards/arm/cornix/` を `boards/jzf/cornix/` に移すだけでは不十分で、`board.yml` と Kconfig 構成も見直す必要がある。

### 2. `board.yml` が必須

Zephyr 4 系では `board.yml` が board 定義の中心になる。

- board 名
- vendor 名
- SoC 情報
- 必要なら revision / variant

このリポジトリでは build 名として `cornix_left` / `cornix_right` を既に使っているため、移行後もこの board 名を維持した方が安全。

関連ファイル:

- `build.yaml`
- `boards/arm/cornix/cornix.zmk.yml`

### 3. `Kconfig.board` は廃止前提で整理する

現行構成では `boards/arm/cornix/Kconfig.board` に以下が入っている。

- `BOARD_CORNIX`
- `BOARD_CORNIX_LEFT`
- `BOARD_CORNIX_RIGHT`

HWMv2 では `Kconfig.board` をそのまま持ち込まない。`Kconfig.<board>`、`Kconfig.defconfig`、共通 `Kconfig` の役割を分けて再構成する必要がある。

### 4. split keyboard は手動移行前提

Cornix は左右別 board の split keyboard なので、自動変換より手動で詰める前提で考える。

特に先に決めるべき点:

- `cornix_left` / `cornix_right` を独立 board として維持するか
- variant で表現するか
- upstream の `zephyr4` ブランチ実装にどこまで合わせるか

将来の取り込みや差分最小化を考えると、`フォーク元 zephyr4 ブランチの表現に合わせる` のが最も安全。

### 5. `BOARD_CORNIX` の扱いを先に決める

このリポジトリでは `BOARD_CORNIX` を共通フラグとして使っている。

該当箇所:

- `boards/arm/cornix/pinmux.c`
- `boards/arm/cornix/Kconfig.defconfig`
- `boards/arm/cornix/cornix.conf`
- `boards/arm/cornix/cornix_right_defconfig`

HWMv2 では通常、board ごとのシンボルは `BOARD_CORNIX_LEFT` / `BOARD_CORNIX_RIGHT` 側が中心になる。したがって次のどちらにするかを先に決める必要がある。

- `BOARD_CORNIX` を廃止し、左右 board 条件へ置き換える
- `BOARD_CORNIX` を独自共通シンボルとして残す

ここを曖昧にしたまま移行すると、Kconfig と C コードが崩れやすい。

### 6. nRF52840 の DCDC / NFC ピン設定を見落とさない

Cornix は `P0.09` / `P0.10` をマトリクスで使っているため、NFC ピンの扱いに注意が必要。

関連ファイル:

- `boards/arm/cornix/cornix_left_common.dtsi`
- `boards/arm/cornix/cornix_right.dts`

また、現状は `boards/arm/cornix/Kconfig` で DCDC を選択しているが、Zephyr 4 系では devicetree 側へ寄る設定がある。HWMv2 移行時に合わせて見直す。

### 7. shield 側は原則そのまま、board 側に集中する

このリポジトリでは `zephyr/module.yml` に `board_root: .` が既にあるため、board 探索ルート自体は揃っている。

そのため、主な作業対象は board 定義側になる。

対象の中心:

- `boards/arm/cornix/`

大きな変更が不要な可能性が高い:

- `boards/shields/cornix_indicator/`

### 8. ドキュメントや補助メモの古いパスも後で必ず直す

現状、`boards/arm/cornix` を参照する記述が README やメモに残っている。

主な対象:

- `README.md`
- `cornix-zmk-notes.md`

コードだけ直してドキュメントを残すと、後から作業する人が誤ったパスを参照しやすい。

## このリポジトリで先に確認しておくべきもの

- `build.yaml`
- `zephyr/module.yml`
- `boards/arm/cornix/Kconfig.board`
- `boards/arm/cornix/Kconfig.defconfig`
- `boards/arm/cornix/Kconfig`
- `boards/arm/cornix/cornix_left_defconfig`
- `boards/arm/cornix/cornix_right_defconfig`
- `boards/arm/cornix/cornix_left.dts`
- `boards/arm/cornix/cornix_right.dts`
- `boards/arm/cornix/cornix.dtsi`
- `boards/arm/cornix/pinmux.c`
- `boards/arm/cornix/cornix.zmk.yml`

## 推奨する移行順

1. フォーク元 `zephyr4` ブランチの Cornix 実装を基準として差分を洗い出す。
2. `board.yml` と board 名の表現方針を先に確定する。
3. `Kconfig.board` 廃止後の Kconfig 分割方針を決める。
4. `BOARD_CORNIX` を残すか廃止するかを決め、参照箇所を揃える。
5. DTS / dtsi / pinctrl / CMake を HWMv2 構造へ移す。
6. DCDC / NFC ピンなど Zephyr 4 系で扱いが変わる点を devicetree 基準で見直す。
7. `build.yaml` と ZMK metadata を確認し、`cornix_left` / `cornix_right` の build 名を維持する。
8. README とメモ類の旧パス参照を更新する。
9. 毎回 pristine build で `cornix_left`、`cornix_right`、`settings_reset` を確認する。

## 検証時の注意

- board 定義変更はキャッシュに引っ張られやすいため、基本は pristine build で確認する。
- 少なくとも以下は毎回確認する。
  - `cornix_left`
  - `cornix_right`
  - `settings_reset`
- board 名や artifact 名を変えると、既存の CI や README 手順にも影響が出る。

## 参照元

- Zephyr Board Porting Guide
  - https://docs.zephyrproject.org/latest/hardware/porting/board_porting.html
- Zephyr Application Development: custom board / board root 関連
  - https://docs.zephyrproject.org/latest/develop/application/index.html
- ZMK: Introducing Zephyr 4.1
  - https://zmk.dev/blog/2025/12/09/zephyr-4.1

