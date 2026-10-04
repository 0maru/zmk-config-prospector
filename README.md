# Cornix + Prospector Scanner

保存済みのVial配列をCornix左側に持たせ、Prospectorを状態表示専用にするZMK設定です。Prospectorがなくても、CornixからPCへUSBまたはBluetoothで入力できます。

```text
Cornix右 ── Bluetooth ── Cornix左 ── USB / Bluetooth ── PC
                             └── 状態を送信 ── Prospectorの画面
```

Prospectorはキー入力を中継しません。USBは給電に使用し、Cornixの電池残量・レイヤーなどを表示します。表示のためのペアリングは不要です。

## ファームウェア

GitHub Actionsの **Build ZMK firmware** がPR・mainへのpush・手動実行でビルドします。成功した実行の **firmware** アーティファクトをダウンロードしてください。

| ファイル | 書き込む機器 |
| --- | --- |
| `cornix_left_standalone.uf2` | Cornix左：配列とPCへの接続を担当 |
| `cornix_right_peripheral.uf2` | Cornix右：左側へキー操作を送信 |
| `prospector_scanner.uf2` | Prospector：状態表示専用 |
| `cornix_settings_reset.uf2` | Cornix左右の設定リセット用（共通） |
| `prospector_settings_reset.uf2` | Prospectorの設定リセット用 |

依存するZMK・Cornix v3.0.0・Prospector Scannerモジュールv2.2.3と再利用workflowは、コミットを固定しています。Cornix v3.0.0に不足するZMK対応フラグは、修飾付きCornixターゲットかつNVS使用時に限って補っています。

## 受信機構成からの初回移行

現在の配列を保存し、機器ごとにファイルを確認して書き込んでください。設定リセットはBluetoothの接続情報とStudioで保存した設定を消します。

1. ProspectorをUSBから外し、Cornix左右の電源を切ります。
2. Cornixの片側をUSB接続し、RESETを素早く2回押してUF2ドライブを表示します。
3. `cornix_settings_reset.uf2` をコピーし、自動再起動後にもう一度RESETを素早く2回押します。
4. 左には `cornix_left_standalone.uf2`、右には `cornix_right_peripheral.uf2` をコピーします。反対側も手順2–4で移行します。
5. 左右をONにして、左をPCへUSB接続します。左右のキーとノブの動作を確認します。
6. 無線で使う場合は、左のUSBを外してPCのBluetooth設定から `Cornix` をペアリングします。以前の同名の登録が残っている場合は削除してから登録し直します。
7. ProspectorをUF2モードにして `prospector_settings_reset.uf2` を書き込みます。再びUF2モードにして `prospector_scanner.uf2` を書き込みます。
8. Cornixが動作中なら、Prospectorが送信された状態を検出して表示します。

Cornixはno-SoftDevice配置（アプリ開始 `0x1000`）、ProspectorはXIAOの標準配置（アプリ開始 `0x27000`）です。復旧用SoftDeviceや異なる機器用UF2を混ぜないでください。

## 配列と普段の更新

保存済み `cornix.vil` の10レイヤーを移植しています。詳しくは [KEYMAP.md](KEYMAP.md) を参照してください。

- 左親指、外側から：レイヤー3・Command・Enter
- 右親指、内側から：Space・レイヤー1・レイヤー2
- 左ノブ：音量、右ノブ：スクロール
- Studio解除：右ノブを押しながら左上のTab

`config/cornix.keymap` を編集してビルド後、**左の `cornix_left_standalone.uf2` だけ**を書き換えます。通常の配列変更では右側やProspectorの再書き込み、設定リセットは不要です。

ZMK Studioを使う場合はCornix左側をUSB接続します。Studioで保存した配列はファームウェア内の配列より優先されます。

## 画面と状態送信

beekeebのXIAO nRF52840とWaveshare 1.69インチLCDに合わせ、タッチと照度センサーを無効化し、明るさ50%にしています。Scannerの標準画面を使い、10レイヤーを表示します。

`config/prospector_scanner.conf` が画面設定、`config/cornix_left_standalone.conf` がCornix側の状態送信設定です。状態送信は入力通信とは別なので、画面の更新に遅れがあってもキー入力はCornixからPCへ直接送られます。

## 参照元

- [Cornix](https://github.com/hitsmaxft/zmk-keyboard-cornix)
- [Prospector Scanner](https://github.com/t-ogura/zmk-config-prospector)
- [Scanner・状態送信モジュール](https://github.com/t-ogura/prospector-zmk-module)
- [Prospectorハードウェア](https://github.com/carrefinho/prospector)
- [beekeeb組立ガイド](https://docs.beekeeb.com/build-guide/prospector-zmk-dongle-photo-build-log-and-firmware)

キーマップはCornixモジュールのMITライセンスの標準キーマップを元に、保存済みVial設定を移植しています。Scanner設定は上記Prospector Scannerの非タッチ構成を参考にしています。
