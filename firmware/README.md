# Scanner Pocket ファームウェア保管庫

このディレクトリの `.uf2` は **git 管理外**です（リポジトリ直下の `.gitignore` に `*.uf2`。
カラー版 `zmk-config-prospector` も同じ方式で、バイナリは置かず文書だけを残しています）。
このファイルがその文書にあたるので、バイナリを消したり移したりしたら合わせて更新してください。

**ファイル名の規則**: `scanner_pocket_<版>_<モジュール側コミット>_<LCD配線>_nav<ボタン>.uf2`

すべて `.uf2` は 2026-09-27 に再ビルドしたものです（ソースは各コミット時点のまま）。

---

## 一覧

| # | ファイル | 版 | 配線 | コア | 実機実績 |
|---|----------|----|------|------|----------|
| 1 | `scanner_pocket_v0.1_c25d43f_D10-D9-D8_navD6.uf2` | v0.1（タグ有） | D10/D9/D8・nav D6 | v2.1 系 | v0.1 当時に動作 |
| 2 | `scanner_pocket_newpin_f9a6372_D4-D5-D6_navD0.uf2` | タグ無（移植前） | **D4/D5/D6・nav D0** | v2.1 系 | 2026-05-27 にこの構成で動作 |
| 3 | `scanner_pocket_core2.2.3_1b8ef72_D4-D5-D6_navD0.uf2` | タグ無（移植後） | **D4/D5/D6・nav D0** | v2.2.3 共有コア | **未検証**（不調報告あり） |
| D | `scanner_pocket_newpin_f9a6372_D4-D5-D6_navD0_DEBUG-usblog.uf2` | #2 + USB ログ | **D4/D5/D6・nav D0** | v2.1 系 | 診断用 |

> 現在の実機配線は #2 / #3 の **D4/D5/D6・nav D0** です。#1 は旧配線なので、いまの基板では映りません。

---

## 1. v0.1

- **モジュール**: `prospector-zmk-module` `c25d43f` — タグ `scanner-pocket-v0.1`（2026-01-29 10:24:11 +0900）
  `feat(scanner_pocket): Add navigation button and keyboard list screen`
- **config**: `zmk-scanner-pocket` `61be5ec` — タグ `v0.1`（2026-01-29 10:24:21 +0900）
- **配線**: SPI1 D10=MOSI(P1.15) / D9=SCLK(P1.14) / D8=CS(P1.13, Active HIGH)、ナビボタン D6、**D3 を GPIO hog で GND 代わりに使用**
- **サイズ**: 673,792 B / FLASH 336,840 B (41.74%) / RAM 119,796 B (45.70%)
- **sha256**: `5ebfca83d211db6af4397dc3fa37e0fe7d2c0bf111110186fd485413b52ab12e`
- 再現: `git -C modules/prospector-zmk-module checkout scanner-pocket-v0.1 && git checkout v0.1`

## 2. 移植前・新配線（その配線で最後に正常だった版）

- **モジュール**: `prospector-zmk-module` `f9a6372`（2026-08-09 20:55:14 +0900、`feature/scanner-pocket-widgets`）
  `WIP: Scanner Pocket - new pinout, scan duty Kconfig, name/channel fixes`
  ※ コミット日は 2026-08-09 だが、**コード自体は 2026-05-27 に書かれ動作確認された内容**。
  8/9 のセッションで未コミットのまま残っていたものを確定させただけ。
- **config**: `zmk-scanner-pocket` `0cfd9e1`（= `main`）
- **配線**: SPI1 D4=MOSI(P0.04) / D5=SCLK(P0.05) / D6=CS(P1.11, Active HIGH)、ナビボタン D0、GND は XIAO の GND ピン直結（D3 hog 廃止）
- **サイズ**: 674,304 B / FLASH 336,920 B (41.75%) / RAM 119,796 B (45.70%)
- **sha256**: `96b11754455d3d1e7d2407b374b2be92c696a0d20a0256b1dc73a5d667814855`
- **中身**: v2.1 系スキャナコア（コールバック方式）。新ピン配置、スキャンデューティ Kconfig、
  BLE アドレスローテーション対策、SCAN_RSP 遅延名の修正を含む。
- 再現: `git -C modules/prospector-zmk-module checkout f9a6372`（config は `main` のまま）

## 3. 移植後・v2.2.3 共有コア（未検証）

- **モジュール**: `prospector-zmk-module` `1b8ef72`（2026-08-09 21:10:55 +0900、`feature/scanner-pocket-v2.3`）
  `Rebuild Scanner Pocket on the v2.2.3 shared scanner core`
- **config**: `zmk-scanner-pocket` `0cfd9e1`（= `main`）
- **配線**: #2 と同一（D4/D5/D6・nav D0）
- **サイズ**: 675,840 B / FLASH 337,704 B (41.85%) / RAM 121,972 B (46.53%)
- **sha256**: `52c23f642ef686c353a3ae56668ee5aebe3239910dca5a54721f00fb410f36e3`
- **中身**: スキャナコアを `src/scanner_core.c` へ昇格させた v2.2.3 系に移植。
  データ受け渡しをコールバックからポーリング（`pull_from_core()`）へ変更。
- **状態**: ビルドのみ確認、**実機未検証**。不調の報告あり。
  疑わしい箇所はコアの `selected_keyboard` の噛み合わせ
  （全キーボードのタイムアウト後にスロットが陳腐化して「Scanning...」から復帰しない問題は
  カラー版で 2026-08-31 に修正済みだが、この版はそれより前）。
- 再現: `git -C modules/prospector-zmk-module checkout feature/scanner-pocket-v2.3`

---

## D. 診断用: #2 + USB CDC シリアルログ

通常版は `CONFIG_LOG=n` かつ `CONFIG_CONSOLE` 未設定で**完全に無言**（LED も一切駆動しない）ため、
マイコンが生きているかを確認できない。その診断用に #2 へログを足したもの。

- **ソース**: #2 と同一（モジュール `f9a6372`、config `0cfd9e1`）。差分は Kconfig のみでコード変更なし。
- **サイズ**: 839,168 B / FLASH 419,388 B (51.97%) / RAM 135,796 B (51.80%)
- **sha256**: `c7b843922984bfa13ced393ebbfb490c9522778ab32a1489c6789a1139ae16ab`
- **追加設定**: `debug_log.conf`（このディレクトリに保存）+ `-DSNIPPET=zmk-usb-logging`

```bash
git -C modules/prospector-zmk-module checkout f9a6372
rm -rf build && .venv/bin/west build -b xiao_ble/nrf52840 -s zmk/app -- \
  -DSHIELD=scanner_pocket \
  -DZMK_CONFIG="/home/ogu/workspace/prospector/zmk-scanner-pocket/config" \
  -DSNIPPET="zmk-usb-logging" \
  -DEXTRA_CONF_FILE="/home/ogu/workspace/prospector/zmk-scanner-pocket/firmware/debug_log.conf"
```

**ログの読み方**: USB 接続で CDC ACM シリアルポートとして現れる（Linux/WSL は `/dev/ttyACM0`、
Windows は COMx）。ボーレートは CDC なので任意。

```bash
screen /dev/ttyACM0 115200      # or: minicom -D /dev/ttyACM0
```

**重要な制約**: ログは DEFERRED モードで、ZMK が `LOG_PROCESS_THREAD_STARTUP_DELAY_MS=1000` を
設定している。USB 列挙を待ってからバッファを吐く設計なので起動直後のログも取れるが、
**起動 1 秒以内にクラッシュするとバッファごと失われる**。その場合は下記へ切り替える。

- **UART 版**: D1/D2 が空いているので物理 UART に出せる（`CONFIG_LOG_MODE_IMMEDIATE=y` と併用で
  最初の 1 行から取れる）。USB-TTL アダプタが必要。
- **RTT 版**: `CONFIG_ZMK_RTT_LOGGING=y`。SWD プローブ（J-Link 等）が必要。

なおこの Zephyr 4.1 では fatal error が `arch_system_halt` になる（`RESET_ON_FATAL_ERROR` が無い）ため、
**あらゆるクラッシュがフリーズとして見える**。無反応 = 電源やクロックの問題とは限らない。

---

## 共通のビルド条件

3つすべて同じワークスペースで、同じ素材から 2026-09-27 にビルドしています。

| 項目 | 値 |
|------|-----|
| board | `xiao_ble/nrf52840` |
| shield | `scanner_pocket` |
| zmk | `354cff9c36b49eee6abbb8a61e6b927539aebbf2`（2026-01-17） |
| zephyr | `ec36516990d40355238db3049bc1709191f99b4e` v4.1.0+zmk-fixes（2026-01-07） |
| lvgl | `f1db87ee98f1810328a8419572fa42a3b5f352ae` |
| Zephyr SDK | 0.16.5 |

このワークスペースは 2026-01 以降 `west update` していないため、ZMK/Zephyr は v0.1 当時とほぼ同じ時点です。
`west update` を実行すると以後の再ビルドは別の ZMK で組まれ、上記と一致しなくなります。

```bash
cd /home/ogu/workspace/prospector/zmk-scanner-pocket
# 目的のコミットを checkout してから
rm -rf build && .venv/bin/west build -b xiao_ble/nrf52840 -s zmk/app -- \
  -DSHIELD=scanner_pocket \
  -DZMK_CONFIG="/home/ogu/workspace/prospector/zmk-scanner-pocket/config"
```

ログ付きが必要な場合は `-DSNIPPET=zmk-usb-logging` を追加（`config/scanner_pocket.conf` は
`CONFIG_LOG=n` なので、併せて `CONFIG_LOG=y` の overlay が必要）。
