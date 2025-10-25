# Agent Guidelines

- 編集が許可されているのは `config/KobitoKey_QWERTY/` 配下のみ。忘れずに守ること。
- PMW3610 ドライバは `badjeff/zmk-pmw3610-driver` を参照する。モジュール更新時は `west update` を実行。
- 変更後は `just build KobitoKey_left` と `just build KobitoKey_right` でビルドを必ず確認する。他ターゲットの指示がある場合は追加で実行すること（例: `just build settings_reset`）。
- Split 構成では右手が Central、左手が Peripheral。役割を変える際は `.conf` の `CONFIG_ZMK_SPLIT_ROLE_*` 周辺を確認。
