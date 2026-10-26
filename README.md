# Cornix — USB / Bluetooth直接接続

保存済みのVial配列をCornix左側に持たせ、PCへUSBまたはBluetoothで直接接続するZMK設定です。Prospectorの状態送信・表示モジュールは使用しません。

```text
Cornix右 ── Bluetooth ── Cornix左 ── USB / Bluetooth ── PC
```

左側が親機として配列とPCへの接続を担当し、右側は左側へキー・ノブ操作を送ります。USBで入力するときは**左側**をPCへ接続してください。左右間はBluetoothで通信します。

## ファームウェア

GitHub Actionsの **Build ZMK firmware** がPR・mainへのpush・手動実行でビルドします。成功した実行の **firmware** アーティファクトをダウンロードしてください。

| ファイル | 書き込む機器 |
| --- | --- |
| `cornix_left_standalone.uf2` | Cornix左：配列とPCへの接続を担当 |
| `cornix_right_peripheral.uf2` | Cornix右：左側へキー・ノブ操作を送信 |
| `cornix_settings_reset.uf2` | Cornix左右の設定リセット用（共通・通常更新では使用しない） |

依存するZMK・Cornix v3.0.0と再利用workflowは、コミットを固定しています。Cornix v3.0.0に不足するZMK対応フラグは、修飾付きCornixターゲットかつNVS使用時に限って補っています。

## 既存の左親機構成から更新

このリポジトリの従来の「Cornix + Prospector Scanner」構成も左側が親機です。状態送信を取り除く今回の更新では、**左の `cornix_left_standalone.uf2` だけ**を書き換えます。右側の再書き込みや設定リセットは不要です。Prospectorは取り外せます。

1. Cornix左をUSB接続し、RESETを素早く2回押してUF2ドライブを表示します。
2. `cornix_left_standalone.uf2` をコピーし、自動再起動を待ちます。
3. 右側の電源をONにして、左右のキーとノブが入力できることを確認します。
4. 無線で使う場合は左のUSBを外します。既存のBluetooth登録をそのまま使えます。未登録ならPCのBluetooth設定で `Cornix` をペアリングしてください。

Bluetoothの接続情報とStudioの保存設定は通常の書き換えで保持されます。Studioで保存した配列は、ファームウェア内の配列より優先されます。

## 受信機が親機の構成から移行・ペアリングの修復

以前のファームウェアで受信機が親機だった場合や、左右のペアリングが成立しない場合に使用します。**設定リセットはBluetoothの接続情報とStudioで保存した設定を消します。** 必要な配列を先に保存してください。

1. 受信機を外し、Cornix左右の電源を切ります。
2. Cornixの片側をUSB接続し、RESETを素早く2回押してUF2ドライブを表示します。
3. `cornix_settings_reset.uf2` をコピーし、自動再起動を待ってUSBを外します。
4. 反対側も手順2–3でリセットします。**両側のリセットが終わってから**通常ファームウェアを書き込みます。
5. 再びUF2モードにして、左には `cornix_left_standalone.uf2`、右には `cornix_right_peripheral.uf2` をコピーします。
6. 左右を同時に再起動し、左をPCへUSB接続して左右のキーとノブを確認します。
7. PCに残った以前の `Cornix` のBluetooth登録を削除します。左のUSBを外し、PCのBluetooth設定から `Cornix` を登録し直します。

Cornixはno-SoftDevice配置（アプリ開始 `0x1000`）です。これらのUF2はCornix専用です。

## 配列と普段の更新

保存済み `cornix.vil` の10レイヤーを移植しています。詳しくは [KEYMAP.md](KEYMAP.md) を参照してください。

- 左親指、外側から：レイヤー3・Command・Enter
- 右親指、内側から：Space・レイヤー1・レイヤー2
- 左ノブ：音量、右ノブ：スクロール
- Studio解除：右ノブを押しながら左上のTab

`config/cornix.keymap` を編集してビルド後、**左の `cornix_left_standalone.uf2` だけ**を書き換えます。通常の配列変更では右側の再書き込みや設定リセットは不要です。

ZMK Studioを使う場合はCornix左側をUSB接続します。`config/cornix_left_standalone.conf` でStudioのロックと右側の電池残量取得を設定しています。

## 参照元

- [Cornix](https://github.com/hitsmaxft/zmk-keyboard-cornix)
- [ZMK：左右の接続と親機の役割](https://zmk.dev/docs/features/split-keyboards)
- [ZMK：ペアリングのリセット](https://zmk.dev/docs/troubleshooting/connection-issues)

キーマップはCornixモジュールのMITライセンスの標準キーマップを元に、保存済みVial設定を移植しています。
