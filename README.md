# KobitoKey_QWERTY

Layer 0 QWERTY
<img width="1280" height="690" alt="Image" src="https://github.com/user-attachments/assets/ef0797b7-a63f-4632-912d-9b5d0115769f" />

Layer 1 NUMBER & ARROW
<img width="1280" height="690" alt="Image" src="https://github.com/user-attachments/assets/d6347b3c-a238-4278-bacd-e58195774d0e" />

Layer 2 Bluetooth & FUNCTION
<img width="1280" height="690" alt="Image" src="https://github.com/user-attachments/assets/f1f7cc93-fbd8-4a98-84ea-c8c36ad3952d" />

Layer 3 AUTO MOUSE
<img width="1280" height="690" alt="Image" src="https://github.com/user-attachments/assets/2efe5275-e460-41bc-ae45-0c0665435268" />

## 構成メモ
- PMW3610 用ドライバは [`badjeff/zmk-pmw3610-driver`](https://github.com/badjeff/zmk-pmw3610-driver) を参照するよう `config/west.yml` を更新。`west update` で最新を取得する。
- センサー回転モジュール（`zmk-feature-sensor_rotation`）は従来どおり利用するが、manifest から管理しやすいよう整理済み。
- `build.yaml` に `settings_reset` ビルドを追加し、GitHub Actions やローカルビルドのマトリクスを moNa2 構成に合わせて整備。左右ファーム刷新後は `settings_reset` をフラッシュして BLE 設定を初期化する。
- 左右の役割は右手が Central（`CONFIG_ZMK_SPLIT_ROLE_CENTRAL` 有効）／左手が Peripheral（中央側は `CONFIG_ZMK_SPLIT_ROLE_CENTRAL` 未設定）となるよう配置。
- Split 構成が明示的に有効になるよう、左右とも `CONFIG_ZMK_SPLIT=y` と `CONFIG_ZMK_SPLIT_BLE=y` を設定。左手は BLE 接続安定化のため接続間隔 (`CONFIG_BT_PERIPHERAL_PREF_*`) と USB 無効化を追加し、右手で ZMK Studio などの中央機能を扱う。
