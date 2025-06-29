# ZMK多ボタンマウス設計仕様書

## 1. ハードウェア基本仕様

### 1.0 デバイス概要

- 12個のボタン、1個のホイールエンコーダ、マウスセンサPMW3610を搭載した多ボタンマウス
- Product name: Snugon MO1
- フォルダ作成やファイル作成でSnugon MO1を使用する場合、"snugon_mo1"とする。
- 開発者名: Shota Nishide
- ブランド名: Snugon

### 1.1 ボタン構成

- **物理ボタン数**: 12個
- **ボタン配置**:
    - 人差し指: 2個（左クリック + 追加1個）
    - 中指: 2個（右クリック + 追加1個）
    - 親指: 2個（サイドボタン）
    - 薬指: 2個
    - 小指: 2個
    - ホイール押し込み: 1個（中クリック）
    - ホイール手前: 1個
- **初期割り当て**:
    - ボタン1: 左クリック（&mkp LCLK）
    - ボタン2: 右クリック（&mkp RCLK）
    - ボタン3: 中クリック（&mkp MCLK）
    - ボタン4: Enter（&kp ENTER）- 親指指先側
    - ボタン5: Esc（&kp ESC）- 親指根本側
    - ボタン6-12: 未割り当て（ユーザー設定用）

### 1.2 ホイール

- **個数**: 1個
- **機能**:
    - 縦スクロール
    - 押し込みボタン（中クリック）
- **チルト機能**: なし
- **レイヤー別機能切り替え**: 対応可能

### 1.3 センサー

- **型番**: PixArt PMW3610
- **選定理由**: 省電力（16-30μA）、ワイヤレス用途に最適
- **DPI設定**:
    - 6段階: 400 / 800 / 1200 / 1600 / 2000 / 2400
    - デフォルト: 1600 DPI
- **DPI機能**:
    - DPI UP/DOWN: オプション機能（カスタムビヘイビア）
    - DPI 0（一時停止）: オプション機能（カスタムビヘイビア）

## 2. MCU・通信仕様

### 2.1 マイクロコントローラー

- **モジュール**: Seeed XIAO nRF52840
- **選定理由**:
    - 技適取得済み
    - 入手性良好（国内代理店あり）
    - 価格: 約1,500円/個
    - ZMK対応実績あり

### 2.2 I/O拡張

- **IC**: MCP23017（16ピンI/Oエクスパンダー）
- **用途**: 12個のボタン入力処理
- **接続**: I2C（SDA、SCL）
- **パッケージ**: SSOP-28推奨

### 2.3 通信方式

- **接続方式**: Bluetooth + USB両対応
- **Bluetooth仕様**:
    - BLE 5.0
    - マルチペアリング: 5台
    - プロファイル切り替え: ダイレクト選択可能
- **USB仕様**:
    - USB 2.0 Full Speed
    - VID: 0x385B（独自取得済み）
    - PID: 0x0200
- **出力切り替え**: &out OUT_USB/OUT_BLE/OUT_TOG

## 3. 電源・バッテリー仕様

### 3.1 バッテリー

- **種類**: リチウムポリマー電池
- **容量**: 250mAh（仮決定、筐体サイズに応じて変更可）
- **想定稼働時間**: 約1-2ヶ月（PMW3610使用時）

### 3.2 充電

- **方式**: USB Type-C
- **充電中の動作**: 使用可能
- **実装方法**: Type-Cブレークアウトボード経由

### 3.3 省電力設定

- **スリープ移行時間**: 60秒（無操作時）
- **ディープスリープ**: 対応

### 3.4 LED表示

- **種類**: RGB LED 1個
- **バッテリー残量表示**:
    - 0-20%: 赤
    - 20-50%: オレンジ
    - 50-80%: 黄色
    - 80-100%: 緑
- **動作**:
    - 常時消灯（省電力）
    - ボタン押下時のみ3秒間表示
    - 表示トリガー: オプション機能（ユーザー割り当て）

## 4. ソフトウェア設計

### 4.1 レイヤー構成

- **総レイヤー数**: 32
- **レイヤー0**: 基本レイヤー（最小限定義）
    - 左/右/中クリック、Enter、Esc
    - 残り7個は未割り当て
- **レイヤー1-31**: 完全に未定義（ユーザー設定用）

### 4.2 オプション機能（ビヘイビア）

- **DPI関連**:
    - &dpi_up: DPI増加
    - &dpi_down: DPI減少
    - &dpi_zero: DPI一時停止（押下中のみ）
- **Bluetooth関連**:
    - &bt BT_SEL 0-4: プロファイル1-5切替
    - &bt BT_CLR: ペアリングクリア
    - &bt BT_NXT/PRV: 次/前のプロファイル
- **その他**:
    - &battery_check: バッテリー残量表示

### 4.3 キーマップ配布方針

- レイヤー0のみ最小限定義
- サンプル設定を別途文書化
- ユーザーによる完全カスタマイズを想定

### 4.4 カスタマイズツールへの対応

- Keyap Editorに対応
- ZMK Studioに対応
    - 公式ドキュメント：https://zmk.dev/docs/features/studio

## 5. ハードウェア設計詳細

### 5.1 ピン配置

#### MCU直接接続（Seeed XIAO）:

- **SPI（センサー用）**:
    - D8 (P1.11): SPI_SCK
    - D9 (P1.12): SPI_MOSI
    - D10 (P1.13): SPI_MISO
    - D7 (P1.08): SPI_CS
    - 空きGPIO: センサーIRQ（新規追加）
- **I2C（MCP23017用）**:
    - SDA: 1ピン
    - SCL: 1ピン
- **エンコーダー**:
    - A相: 1ピン
    - B相: 1ピン
- **その他**:
    - RGB LED: 1ピン（WS2812B）
    - バッテリー電圧監視: 1ピン（ADC）

#### MCP23017経由:

- 12個のボタン入力（GPA0-7、GPB0-3使用）

### 5.2 基板設計変更点

- **流用**: 既存QMK版基板（RP2040版）をベースに改修
- **主な変更**:
    - MCU部: RP2040 → Seeed XIAO nRF52840
    - ボタン配線: MCU直接 → MCP23017経由
    - センサー: PMW3360 → PMW3610（IRQピン追加）
    - 新規追加: バッテリーコネクタ（JST-PH等）
    - USB: Type-Cブレークアウトボード経由で延長

### 5.3 アンテナ配置

- XIAOのアンテナ部を基板内側（USB端子の反対側）に配置
- 金属部品から10mm以上のクリアランス確保
- アンテナ周辺の銅箔除去

## 6. 認証・法的要件

### 6.1 必要な認証

- **PSEマーク**: 必要（リチウムイオン蓄電池搭載のため）
- **技適**: Seeed XIAOで取得済み

### 6.2 ライセンス対応

- **使用するセンサードライバー**: inorichi/zmk-pmw3610-driver
    - ポインティングデバイスについては公式ドキュメント(https://zmk.dev/docs/features/pointing)も参照
- **ライセンス**: MIT（推定）
- **対応**: README.mdまたは製品内にライセンス表記を含める

markdown

```markdown
This product uses PMW3610 driver by inorichi
https://github.com/inorichi/zmk-pmw3610-driver
```

### 6.3 公開・非公開の切り分け

- **公開部分**:
    - 基本的なkeymap設定
    - 最小限のデバイス設定
    - ライセンス表記
- **非公開部分**:
    - センサーの詳細設定
    - 電力最適化パラメータ
    - ハードウェア設計詳細

## 7 デバッグ方法

- **方式**: USBシリアルログ
- **設定**: CONFIG_ZMK_USB_LOGGING=y

## 8. 今後の拡張性

### 8.1 ハードウェア

- バッテリー容量は筐体に合わせて変更可能
- 将来的なチップ直接実装への移行パス確保

### 8.2 ソフトウェア

- 31個の未使用レイヤーで高い拡張性
- カスタムビヘイビアによる機能追加
- 将来的なZMK Studio対応の可能性

## 9. リスク管理

### 9.1 技術的リスク

- センサードライバーのメンテナンス依存
- ZMK本体の破壊的変更への対応

### 9.2 対策

- inorichiドライバーの定期的な動作確認
- 将来的なZephyr公式ドライバーへの移行検討