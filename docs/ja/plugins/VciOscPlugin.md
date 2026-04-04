# 🎮 VCI OSC テロップ (VciOscPlugin.py)

> 📥 **[プラグインをダウンロード](https://github.com/miyumiyu/TeloPon/releases/download/plugins-v1.3/VciOscPlugin.py)**

このプラグインは、AIが出力したテロップを **OSC（OpenSound Control）** プロトコル経由で **VirtualCast の VCI** にリアルタイム送信します。
VirtualCast のバーチャル空間内にテロップを表示させることができます。

---

## 🛠️ 準備

### 必要なもの

1. **VirtualCast**（PC版）がインストール・起動済み
2. VirtualCast の設定で **OSC受信機能** が有効（ポート: 19100）
3. テロップ表示用の **VCI** が VirtualCast 上に配置されていること

### VirtualCast 側の OSC 設定

1. VirtualCast のタイトル画面 → **「VCI」** 設定を開く
2. **「OSC受信機能」** を **「有効」** または **「クリエイターのみ」** に設定
3. 受信ポートが **19100**（デフォルト）になっていることを確認

### VCI 側の受信スクリプト（Lua）

テロップ表示用 VCI に以下のような Lua スクリプトを設定します：

```lua
-- TeloPon からのテロップを受信して表示する
vci.osc.RegisterMethod("/telopon/telop/text", function(sender, name, args)
    local text = args[1]
    vci.assets.SetText("TextBoard", text)
end, {ExportOscType.String})

-- バッジ（感情タグ）を受信する場合
vci.osc.RegisterMethod("/telopon/telop/text/badge", function(sender, name, args)
    local badge = args[1]
    -- バッジに応じた演出など
end, {ExportOscType.String})

-- ウィンドウ種類を受信する場合
vci.osc.RegisterMethod("/telopon/telop/text/window", function(sender, name, args)
    local window = args[1]
    -- ウィンドウ種類に応じた表示切替など
end, {ExportOscType.String})
```

> ⚠️ OSC アドレスはデフォルト `/telopon/telop/text` です。TeloPon 側の設定と一致させてください。

---

## ⚙️ TeloPon 側の設定と使い方

### 1. 操作パネルを開く

TeloPon のメイン画面右側、「拡張機能（プラグイン）」パネルにある **「VCI OSC テロップ」** の **「操作パネル」** ボタンをクリックします。

<img width="400" alt="VCI OSC テロップ設定画面" src="../../images/VciOscPlugin.png" />

### 2. OSC 送信 ON

**「OSC送信 ON」** チェックボックスをオンにすると、テロップが出るたびに自動で VCI に OSC メッセージが送信されます。

### 3. 接続設定

| 設定 | デフォルト値 | 説明 |
|---|---|---|
| **送信先IP** | `127.0.0.1` | VirtualCast が動作しているPCのIPアドレス（同一PCならそのまま） |
| **送信先ポート** | `19100` | VirtualCast の OSC 受信ポート |
| **OSCアドレス** | `/telopon/telop/text` | VCI 側の `RegisterMethod` と一致させる |

### 4. テスト送信

**「テスト送信」** ボタンを押すと、テストメッセージが送信されます。VCI 側でテキストが表示されれば接続成功です。

### 5. 送信内容設定

| 設定 | デフォルト | 説明 |
|---|---|---|
| **📌 TOPICも送信する** | ON | ONの場合「TOPIC \| MAIN」の形式で送信。OFFならMAINのみ |
| **🏷️ バッジも送信する** | OFF | ONの場合、バッジ（感情タグ）を `/address/badge` で別途送信 |

### 6. 閉じる

**「閉じる」** ボタンまたは **×** ボタンで設定パネルを閉じます。設定は自動的に `plugins/vci_osc.json` に保存されます。

---

## 📡 送信される OSC メッセージ

| OSCアドレス | データ型 | 内容 | 例 |
|---|---|---|---|
| `/telopon/telop/text` | String | テロップ本文（TOPIC含む場合あり） | `配信開始 \| てろぽん様が来たぞ！` |
| `/telopon/telop/text/badge` | String | バッジ名（オプション） | `呆れ` |
| `/telopon/telop/text/window` | String | ウィンドウ種類（常に送信） | `window-simple` |

> 💡 テロップ本文の HTML タグ（`<b1>`, `<b2>`, 麻雀牌タグ等）は自動的に除去されてプレーンテキストで送信されます。

---

## ❓ トラブルシューティング

### Q. テスト送信で「❌ 送信失敗」と出る
- 送信先IPとポートが正しいか確認してください
- ファイアウォールがUDPポート19100をブロックしていないか確認してください

### Q. VCI 側でテキストが表示されない
- VirtualCast の OSC 受信機能が有効になっているか確認してください
- VCI の Lua スクリプトの OSC アドレスが TeloPon 側と一致しているか確認してください
- VCI が VirtualCast 上に配置されているか確認してください

### Q. 別のPCの VirtualCast に送信したい
- 送信先IPを VirtualCast が動作しているPCの IP アドレスに変更してください
- 受信側 PC のファイアウォールでポート 19100 (UDP) を許可してください

---

## 📋 技術仕様

| 項目 | 値 |
|---|---|
| プロトコル | OSC over UDP |
| ライブラリ | python-osc (pythonosc) |
| メッセージサイズ上限 | 約4000バイト（VCI側の制限） |
| 対応 | VirtualCast PC版のみ |

---
