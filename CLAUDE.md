# FrostOrtho zmk-config — Claude 向けメモ

FrostOrtho（分割40%キーボード、トラックボール付き）の ZMK 設定リポジトリ。
本家: `imo00o/zmk-config-FrostOrtho` / このリポジトリ: `Ken8712/zmk-config-FrostOrtho`

## ⚠️ 最初に読むもの

**`docs/manual/` に ZMK v0.3 公式ドキュメントの日本語版一式がある。**
`docs/` は `.gitignore` で除外しているため **git には入っていない**（ローカルにのみ存在）。
このリポジトリで作業するときは、まず `docs/manual/README.md` の目次を見て、必要な章を読むこと。

特に **`docs/manual/20-FrostOrtho固有事項.md`** は公式 ZMK との差分をまとめたもので、
これを読まずに公式ドキュメント（zmk.dev）だけを参照すると必ず食い違う。

`docs/manual/` が存在しない環境（別PC、CI、クローン直後）では、
zmkfirmware/zmk の `v0.3-branch` の `docs/docs/` から再生成する。

## バージョン（重要）

| 項目 | 値 | 定義箇所 |
| --- | --- | --- |
| ZMK 本体 | **`cormoran/zmk` の `v0.3-branch+dya`**（公式 v0.3.0 のフォーク = DYA Studio 全部入り） | `config/west.yml` |
| CI ワークフロー | `zmkfirmware/zmk@v0.3.0` の `build-user-config.yml` | `.github/workflows/build.yml` |
| ボード | `seeeduino_xiao_ble`（XIAO nRF52840） | `build.yaml` |
| シールド | `FrostOrtho_L` / `FrostOrtho_R` / `settings_reset` | `build.yaml` |
| Central | **右手（`FrostOrtho_R`）**。`snippet: studio-rpc-usb-uart` は右手のみ | `Kconfig.defconfig` / `build.yaml` |

**ZMK のバージョンを上げるときは `config/west.yml` と `.github/workflows/build.yml` の両方**を更新すること（2か所に分かれている）。

DYA Studio 系モジュール（`cormoran/*` 5個）の機能は **zmk.dev には載っていない**。
`runtime-sensor-rotate` / `mouse_runtime_input_processor` / `battery_history_request` /
`CONFIG_ZMK_BLE_MANAGEMENT*` / `CONFIG_ZMK_SETTINGS_RPC*` / `CONFIG_ZMK_RUNTIME_*` /
`CONFIG_ZMK_SPLIT_RELAY_EVENT` などは各モジュールの README を見る。

## 触るときの注意（詳細は 20-FrostOrtho固有事項.md）

- **レイヤー番号 5 / 6 / 7 が `FrostOrtho_R.overlay` にハードコード**されている
  （5=縦スクロール、6=AML、7=横スクロール）。ZMK Studio でレイヤーを並べ替えると全部ズレる。
- `zip_temp_layer` の `excluded-positions = <6 7 8 20 30 32>` も**キー位置番号**で固定。
  物理レイアウトを変えたら振り直す。
- **`CONFIG_BT_MAX_PAIRED=5` のコメントを外してはいけない。**
  `ZMK_BLE_PROFILE_COUNT = BT_MAX_PAIRED − 周辺機器数` なので 6−1=5 → 5−1=4 に減り、
  keymap の `&bt BT_SEL 4` が使えなくなる。
- SPI の `MOSI` と `MISO` が同じ `P0.04` なのは **PAW3222 の3線式仕様。誤記ではない**ので直さない。
- 右手の `gpio0 9` は列 GPIO ではなく **SPI の CS**。`CONFIG_NFCT_PINS_AS_GPIOS=y` は左右とも外せない。
- **CI の push トリガーは `config/**` 限定**。`build.yaml` や workflow だけ直しても
  ビルドは走らない（`workflow_dispatch` で手動実行するか `config/` も触る）。

## 既知の軽微な不整合（未修正）

- `config/FrostOrtho.keymap` の `#include <behaviors/runtime-sensor-rotate.dtsi>` が2行重複
- `layer_8` の `G` の位置が `&tap_with_alt F` になっている（`G` の打ち間違いと思われる）
- `#define MOUSE 6` によりレイヤーノード名が `MOUSE` ではなく `6` に展開される
- `sensor-bindings` が2個あるがセンサーは `left_encoder` 1個のみ。`rsr_vol` は定義のみで未使用
- `FrostOrtho_L.conf` に Central 専用の `CONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_*` が2行（無視される）
- `CONFIG_ZMK_BATTERY_HISTORY=n` なのに overlay に `battery_history_request.dtsi` の include が残っている
- `zmk-rgbled-widget` は west.yml にあるが `rgbled_adapter` シールド未使用。L 側だけ設定が有効行のまま

## リポジトリ構成

```
config/                     ZMK 設定（CI のトリガー対象）
  west.yml                  ZMK 本体とモジュールの取得元
  FrostOrtho.keymap         キーマップ（ZMK Studio / Keymap Editor が更新）
  boards/shields/FrostOrtho/  dtsi, overlay, conf, Kconfig
build.yaml                  ビルドマトリクス
doc/                        購入者向けビルドガイド・キーマップ図（git 管理下）
docs/manual/                ZMK 日本語マニュアル（git 管理外）
firmware/                   初期ファームウェア
```

## 作業ルール

- コミット作者は `Ken8712 <skyblow.9@gmail.com>`（既存コミットは keymap-editor[bot]）
- `git push` はこの環境から実行できない（認証情報がないため）。コミットまで行い、push はユーザーに依頼する
- 接続フォルダ内で git を実行するとロックファイル（`.git/index.lock` 等）が消せずに残ることがある。
  作業後に `rm -f .git/index.lock .git/HEAD.lock .git/objects/maintenance.lock` と
  `find .git/objects -name 'tmp_obj_*' -delete` で掃除する
