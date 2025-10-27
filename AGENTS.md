# KobitoKey_QWERTY リポジトリメモ（2024-xx-xx 時点）

## 基本情報
- 対象：Seeed XIAO BLE ベースの KobitoKey スプリットキーボード。
- キーレイアウト：40キー（4×10）、レイヤーは QWERTY / Number & Arrow / Bluetooth & Function / Auto Mouse の4枚構成。
- トラックボール（PixArt PMW3610）を左右に搭載し、ZMK の input processor + sensor rotation モジュールで制御。

## ディレクトリ構成の概要

```
.
├── AGENTS.md                    # このメモ
├── README.md                    # レイヤー画像リンクと概要
├── boards/shields/KobitoKey/    # シールド定義とハード寄りの設定
│   ├── KobitoKey.keymap         # ビルドに参照されるキーマップ本体
│   ├── KobitoKey_left/right.overlay  # 左右個別の DT overlay
│   ├── kobitokey.dtsi           # 行列配線・split入力・レイアウト指定
│   ├── KobitoKey-layouts.dtsi   # 物理レイアウト座標
│   ├── Kconfig.*                # シールド用 Kconfig
│   └── KobitoKey.zmk.yml        # ZMK シールドメタ情報
├── build.yaml                   # `just build`/CI 用のビルドマトリクス
├── build.sh                     # just を呼び出すビルドスクリプト
├── config/                      # ZMK Config 直下で編集するファイル群
│   ├── KobitoKey.keymap         # エディタ編集用のキーマップ（boards版と同内容）
│   ├── KobitoKey_left.conf      # 左手ファーム用 Kconfig 追加（日本語コメント付き）
│   ├── KobitoKey_right.conf     # 右手ファーム用 Kconfig 追加（日本語コメント付き）
│   ├── KobitoKey.json           # ZMK Studio 用の物理レイアウト
│   └── west.yml                 # ZMK v0.3 + 追加モジュール2件を取得
└── zephyr/module.yml            # ZMK への board_root 指定
```

## キーマップとレイヤー
- `default_layer`：QWERTY。親指クラスタで `&lt 2 SPACE`（レイヤー2モーメンタリ）、`&lt 1 ENTER`、右小指 `&mt LALT SLASH` などモッドタップを設定。
- `layer1 ("NUMBER")`：数値列と矢印キー、簡易記号。右上にテンキーレイアウト風配置。
- `layer2 ("FUNCTION")`：BT ペアリング（`&bt BT_SEL 0..4`, `&bt BT_CLR` 等）とファンクションキー列、中央 `&kp DEL`。
- `layer3 ("MOUSE")`：トラックボール操作補助。右手にマウスボタン (`&mkp MB1/2/3`)、`&to 0` でベースに戻る。
- Combo：ホーム段内側 (`pos12/13`) で `LANG2`、親指ホーム側 (`pos16/17`) で `LANG1`、ベース段外側 (`pos0/1`) で `ESC`、右手外側 (`pos10/11`) で `TAB` を入力。

## ポインティングデバイス設定
- 左右どちらも `spi0` に PMW3610 をぶら下げ、`irq-gpios=&gpio0 2`。
- 左側：`cpi=200`、`zip_temp_layer` 除外ポジションを設定し、スクロール変換や X/Y 反転を `zip_*` プロセッサで構成。
- 右側：`cpi=600`。`zip_temp_layer` を通じてレイヤー3（Auto Mouse）を一定時間有効化。
- `sensor_rotation` モジュールを利用し左右で ±24°の回転補正。

## Split / BLE 周り
- `kobitokey.dtsi` で `split_inputs` を定義し、右手デバイスのトラックボール入力を中央に転送。
- 左右 `config/KobitoKey_left.conf` / `config/KobitoKey_right.conf`：共通で BLE スプリット有効、左（ペリフェラル）で `CONFIG_ZMK_KEYBOARD_NAME="KobitoKey"`、右（セントラル）で `CONFIG_ZMK_SPLIT_ROLE_CENTRAL=y` やバッテリープロキシを有効化。
    - 左側は USB を無効化（`CONFIG_ZMK_USB=n`）し、右側に `CONFIG_ZMK_USB=y` を追加して USB 接続時もスプリット全体を使用可能に。
    - 右手トラックボールはローカル専用リスナー（`tb_right_local_listener`）で `zip_xy_transform INPUT_TRANSFORM_Y_INVERT` を適用し、Split 経由の左手スクロールには影響させない。
    - 双方とも `CONFIG_ZMK_MOUSE=y`。

## ビルド・依存関係
- `build.sh`：リポジトリルート（`zmk-workspace`）で `just build KobitoKey_left/right/settings_reset` を実行。
- `build.yaml`：GitHub Actions や `just build` マトリクス用に `seeeduino_xiao_ble` + `KobitoKey_left/right/settings_reset` を定義。
- `config/west.yml`：ZMK 本体（v0.3）に加え、`badjeff/zmk-pmw3610-driver` と `hsgw/zmk-feature-sensor_rotation` を追加取得。

## 直近カスタマイズ時に気をつけたい点
- 物理配列 (`KobitoKey-layouts.dtsi`) と `default_transform` の行列が固定幅 (1u) 前提。変更時は両方の整合性を取る。
- キーマップ編集は `config/KobitoKey.keymap` 側を更新する（シールド側は include のみ）。
- トラックボールの CPI や入力プロセッサ設定は左右で異なるため、共通化・調整時はそれぞれの overlay を確認。
- Combo 定義や `zip_temp_layer` の `excluded-positions` はキー順序に依存。キーマップ変更時は再確認が必要。
