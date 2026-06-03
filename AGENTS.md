# AGENTS.md - Pyuron ZMK 構成概要

このファイルは、`zmk-config-Pyuron` リポジトリで作業する AI エージェントや開発者のためのガイドです。ハードウェア仕様、ファームウェア構成、キーマップ編集プロセス、およびデバイスツリーの設定について説明します。

---

## 1. ハードウェア概要
*   **キーボード**: Pyuron (左右分割型のエルゴノミックキーボード)。
*   **MCU (コントローラー)**: 左右ともに Seeeduino Xiao BLE (nRF52840 搭載)。
*   **ポインティングモジュール**: 左右両側に Pixart PMW3610 トラックボールモジュールを搭載。
*   **RGB LED**: アダプター経由で有効化（`rgbled_adapter` シールドを参照）。

---

## 2. 設定とファイル構成

### ビルド設定
*   **[build.yaml](file:///Users/syamgot/projects/syamgot/zmk-config-Pyuron/build.yaml)**:
    *   ZMK のビルドターゲットを定義しています。
    *   右手側（親機/Central）: `shield: Pyuron_R rgbled_adapter`。ZMK Studio をサポートするため、`studio-rpc-usb-uart` スニペットが有効になっています。
    *   左手側（子機/Peripheral）: `shield: Pyuron_L rgbled_adapter`。
    *   設定リセット用: `shield: settings_reset` (トラブルシューティング用ターゲット)。

### 依存関係とドライバー
*   **[config/west.yml](file:///Users/syamgot/projects/syamgot/zmk-config-Pyuron/config/west.yml)**:
    *   カスタムトラックボールドライバーを読み込んでいます: `badjeff/zmk-pmw3610-driver` (リビジョン: `main`)。
    *   LED ウィジェットモジュールを読み込んでいます: `caksoylar/zmk-rgbled-widget` (リビジョン: `main`)。

---

## 3. デバイスツリー詳細 (シールド設定)
ファイル配置先: **[config/boards/shields/Pyuron/](file:///Users/syamgot/projects/syamgot/zmk-config-Pyuron/config/boards/shields/Pyuron/)**。

### 入力ルーティングとスプリット設定
*   **[Pyuron.dtsi](file:///Users/syamgot/projects/syamgot/zmk-config-Pyuron/config/boards/shields/Pyuron/Pyuron.dtsi)**:
    *   `split_inputs` 内に `trackball_split_L` を定義。左手側のトラックボールは、ZMK のスプリット周辺入力通信を介して未加工の相対移動データを右手側（親機）へ送信します。
    *   右手側（親機）で動作する `trackball_listener_L` が左手側からのイベントをキャッチし、以下の入力プロセッサーを使用して処理します:
        *   `INPUT_TRANSFORM_Y_INVERT`: Y軸を反転。
        *   `zip_xy_scaler 1 16`: 移動速度をスケーリング。
        *   `zip_scroll_scaler 1 5`: スクロール速度をスケーリング。
        *   `zip_xy_to_scroll_mapper`: スクロール切り替え時に座標をスクロールへマッピング。
    *   `trackball_listener_R` は右手側のトラックボールの動きを直接処理します。

### 左手側 (Left) の設定
*   **[Pyuron_L.overlay](file:///Users/syamgot/projects/syamgot/zmk-config-Pyuron/config/boards/shields/Pyuron/Pyuron_L.overlay)**:
    *   トラックボールセンサー（`compatible = "pixart,pmw3610-alt"` の `trackball_L`）のために `spi0` を設定。
    *   物理トラックボールを `trackball_split_L` に紐づけ、入力データが BLE 経由で送信されるようにします。
*   **[Pyuron_L.conf](file:///Users/syamgot/projects/syamgot/zmk-config-Pyuron/config/boards/shields/Pyuron/Pyuron_L.conf)**:
    *   左手側の Kconfig 設定:
        *   `CONFIG_ZMK_POINTING=y` (ZMK ポインティングドライバーを有効化)
        *   `CONFIG_PMW3610_ALT=y` および `CONFIG_PMW3610_ALT_SWAP_XY=y` (ドライバー有効化およびXY軸のスワップ)

### 右手側 (Right) の設定
*   **[Pyuron_R.overlay](file:///Users/syamgot/projects/syamgot/zmk-config-Pyuron/config/boards/shields/Pyuron/Pyuron_R.overlay)**:
    *   トラックボールセンサー（`trackball_R`）のために `spi0` を設定。
    *   `trackball_listener_R` のステータスを `"okay"` に設定。
*   **[Pyuron_R.conf](file:///Users/syamgot/projects/syamgot/zmk-config-Pyuron/config/boards/shields/Pyuron/Pyuron_R.conf)**:
    *   親機側の Kconfig 設定。デバイス名を `CONFIG_ZMK_KEYBOARD_NAME="Pyuron"` に設定。
    *   ZMK Studio 有効化 (`CONFIG_ZMK_STUDIO=y`)。
    *   RGB LED ウィジェットによるレイヤー色の表示を有効化 (`CONFIG_RGBLED_WIDGET_SHOW_LAYER_COLORS=y`)。

---

## 4. キーマップとレイアウトの編集
*   **[config/Pyuron.json](file:///Users/syamgot/projects/syamgot/zmk-config-Pyuron/config/Pyuron.json)**: キーマップエディターで使用する物理座標のレイアウト定義。
*   **[config/Pyuron.keymap](file:///Users/syamgot/projects/syamgot/zmk-config-Pyuron/config/Pyuron.keymap)**:
    *   挙動マクロ、ホールドタップ挙動、コンボ、および各レイヤーが定義されています。
    *   現在のレイヤー構成:
        *   `0: default_layer` (QWERTY 配列、ホームローモディファイア/ホールドタップ搭載)
        *   `1: layer_1` (F1〜F12 ファンクションキー)
        *   `2: layer_2` (数字、記号、演算子)
        *   `3: layer_3` (ナビゲーション/矢印キー、Esc、Tab 操作)
        *   `4: layer_4` (Bluetooth プロファイルの管理、ブートローダー)
        *   `5: layer_5` (空のプレースホルダーレイヤー)
        *   `6: MOUSE` (マウスボタンクリック MB1/MB2/MB3)
    *   このファイルは、Github Actions と統合された Web ベースの [Keymap-Editor](https://nickcoutsos.github.io/keymap-editor/) を使用して直接編集・更新が行われます。
