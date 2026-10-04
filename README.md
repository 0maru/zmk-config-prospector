# Cornix + Prospector

保存済みのVial配列を使うZMK設定です。Cornix左側を親機にする単体構成と、beekeeb Prospector（XIAO nRF52840）を親機にするドングル構成をビルドできます。

Cornix単体構成：

```text
Cornix右 ── Bluetooth ── Cornix左 ── USB / Bluetooth ── PC
```

ドングル構成：

```text
Cornix左 ── Bluetooth ──┐
                       Prospector ── USB ── PC
Cornix右 ── Bluetooth ──┘
```

ドングル構成ではCornix単体からPCへのUSB/Bluetooth接続は利用できません。単体構成との自動切替はできず、切り替える場合は左側の書き換えと対象機器の接続情報リセットが必要です。

## ビルド

GitHub Actionsの **Build ZMK firmware** がpush・PR・手動実行でビルドします。
成功した実行の **firmware** アーティファクトをダウンロードして展開してください。

| ファイル | 書き込む機器 |
| --- | --- |
| `prospector_cornix.uf2` | ProspectorのXIAO nRF52840 |
| `cornix_left_standalone.uf2` | Cornix左（単体構成の親機） |
| `cornix_left_peripheral.uf2` | Cornix左（ドングル構成の子機） |
| `cornix_right_peripheral.uf2` | Cornix右 |
| `prospector_settings_reset.uf2` | Prospectorの設定リセット用 |
| `cornix_settings_reset.uf2` | Cornix左右の設定リセット用（同じファイルを使用） |

依存するZMK、Cornix v3.0.0、Prospectorモジュールとビルドworkflowはコミットを固定しています。
ProspectorはZephyr 4.1対応の `feat/new-status-screens` を使用しています。
Cornix v3.0.0に不足するZMK対応フラグは、ルートの `Kconfig` で修飾付きCornixターゲットかつNVS使用時に限り補っています。

## 書き込み前の確認

- 現在のキーマップを保存してください。このリポジトリの配列は、保存済みのVial設定から移植しています。詳細は [KEYMAP.md](KEYMAP.md) を参照してください。
- Cornixの現在のファームウェアがRMK/VialかZMKか、UF2ブートローダーへ入れるかを確認してください。
- Cornixはv3.0.0のno-SoftDevice配置（アプリ開始 `0x1000`）を使用します。ProspectorはXIAOの標準配置を使用し、no-SoftDevice用snippetを適用しません。
- 上流の旧復旧ガイドにはSoftDevice復元の記述がありますが、現行Cornixの配置とは異なります。通常の書き込みに復旧用ファイルを混ぜず、ブートローダーに入れない場合は個別に確認してください。
- リセット用UF2はBluetoothのペアリング情報やStudioで保存した設定を消します。通常のキーマップ更新では毎回使う必要はありません。

## Cornix単体構成への書き込み

1. ProspectorをUSBから外します。
2. Cornix左右をそれぞれUF2モードにし、`cornix_settings_reset.uf2` を書き込みます。
3. 再度UF2モードにして、左に `cornix_left_standalone.uf2`、右に `cornix_right_peripheral.uf2` を書き込みます。
4. 左右を再起動します。左をPCにUSBでつなぐか、PCのBluetooth設定で `Cornix` をペアリングします。
5. 左右のキー、ノブ、レイヤー切替を確認します。

通常の配列更新は左の `cornix_left_standalone.uf2` だけを書き換えます。右の再書き込みや設定リセットは不要です。

## ドングル構成の初回書き込み

現在のファームウェアとブートローダーが対応していることを確認してから行います。

1. 各機器をUSBで接続し、RESETを素早く2回押してUF2ドライブを表示します。
2. Prospectorには `prospector_settings_reset.uf2`、Cornix左右には `cornix_settings_reset.uf2` をコピーします。
3. 再び各機器をUF2モードにし、それぞれ対応する本番UF2をコピーします。
4. Cornix左右の電源を切り、ProspectorをPCのUSBに接続します。
5. 左側だけ電源を入れて接続を待ち、次に右側の電源を入れます。画面のバッテリー表示はペアリング順になるためです。
6. 左右の入力、エンコーダー、レイヤー表示、バッテリー表示を確認します。

## キー配列と画面

`config/cornix.keymap` を編集すると、次回のActionsビルドへ反映されます。
保存済みVial設定の10レイヤーを移植しています。左親指は外側からレイヤー3・Command・Enter、右親指は内側からSpace・レイヤー1・レイヤー2です。
左エンコーダーは音量、右エンコーダーはスクロールです。各レイヤー、マクロ、元データとの差分は [KEYMAP.md](KEYMAP.md) を参照してください。
通常のキーマップ変更時は、単体構成なら左側、ドングル構成ならProspectorだけを書き換えます。接続情報のリセットは不要です。

ZMK StudioをUSBで利用できます。使用中の親機（Cornix左またはProspector）を接続し、**右ノブを押しながら左上のTab**でロックを解除してください。
Studioで保存した配列はファームウェア内の初期配列より優先されます。

画面はClassic、明るさ50%固定です。beekeeb版に照度センサーはないため無効化しています。
画面設定は `config/cornix_dongle_adapter.conf` にあります。タッチ操作は設定していません。

## 参照元

- [Cornix](https://github.com/hitsmaxft/zmk-keyboard-cornix)
- [Prospector](https://github.com/carrefinho/prospector)
- [ProspectorのZephyr 4.1対応モジュール](https://github.com/carrefinho/prospector-zmk-module/tree/feat/new-status-screens)
- [beekeeb組立ガイド](https://docs.beekeeb.com/build-guide/prospector-zmk-dongle-photo-build-log-and-firmware)
- [ZMKドングル設定](https://zmk.dev/docs/hardware-integration/dongle)

`config/cornix.keymap` はCornixモジュール内のMITライセンスの標準キーマップから派生しています。
該当するライセンスは `LICENSE` を参照してください。
