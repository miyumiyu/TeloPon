[English](https://github.com/miyumiyu/TeloPon/blob/main/docs/en/extend_plugins/ex-plugins-pack_v1.4.md) | **日本語** | [한국어](https://github.com/miyumiyu/TeloPon/blob/main/docs/ko/extend_plugins/ex-plugins-pack_v1.4.md) | [Русский](https://github.com/miyumiyu/TeloPon/blob/main/docs/ru/extend_plugins/ex-plugins-pack_v1.4.md)

TeloPonをさらに便利にするための「公式拡張プラグイン（連携機能）」の最新版です！
必要な機能のファイル（`.py`）だけをダウンロードして、お使いのTeloPonに追加してください。
**※V2.15b のバージョンから使えるようになります。**

## 📦 収録プラグイン

### 🆕 🎮 VCI OSC テロップ (`VciOscPlugin.py`)

<img width="400" alt="VCI OSC テロップ設定画面" src="https://github.com/miyumiyu/TeloPon/blob/main/images/VciOscPlugin.png?raw=true" />

AIが出力したテロップを **OSC（OpenSound Control）** プロトコル経由で **VirtualCast の VCI** にリアルタイム送信します。
* **VirtualCast連携**: バーチャル空間内にテロップを表示
* **送信先設定**: IP・ポート・OSCアドレスをカスタマイズ可能
* **TOPIC・バッジ送信**: テロップ本文に加えて、TOPICやバッジ（感情タグ）も別アドレスで送信可能
* **テスト送信**: ボタン1つで接続確認

📥 [プラグインをダウンロード](https://github.com/miyumiyu/TeloPon/releases/download/plugins-v1.4/VciOscPlugin.py)  
👉 [詳細な使い方はこちら](../plugins/VciOscPlugin.md)

> 💡 **必要環境**: [VirtualCast](https://virtualcast.jp/)（PC版）でOSC受信機能を有効化してください。

### 🔊 VOICEVOX読み上げ (`voicevox_plugin.py`)

<img width="400" alt="VOICEVOX読み上げ設定画面" src="https://github.com/miyumiyu/TeloPon/blob/main/images/voicevox_plugin.png?raw=true" />

AIが出力したテロップのテキストを **VOICEVOX** の音声合成エンジンでリアルタイムに読み上げます。
* **多彩なキャラクターボイス**: VOICEVOXに登録された全キャラクター＋スタイル（性格）に対応。プルダウンで簡単選択。
* **アイコン表示**: 選択したキャラクターのアイコンが設定画面に表示されます。
* **細かな音声調整**: 話速・音量・音高・抑揚・発話前無音の5パラメータをリアルタイム調整。
* **TOPIC読み上げ**: ONにするとテロップの上段テキスト（TOPIC）も一緒に読み上げます。
* **テロップ表示ディレイ**: 読み上げとOBSテロップ表示のタイミングを揃えられます。
* **自動接続確認**: 起動時にVOICEVOXへの接続を確認。接続できない場合は自動でOFFになります。

📥 [プラグインをダウンロード](https://github.com/miyumiyu/TeloPon/releases/download/plugins-v1.4/voicevox_plugin.py)  
👉 [詳細な使い方はこちら](../plugins/voicevox_plugin.md)

> 💡 **必要環境**: [VOICEVOX](https://voicevox.hiroshiba.jp/) をインストールし、TeloPonと同時に起動してください。

### 💬 Discordリアルタイム連携 (`discord_integration.py`)
Discordサーバーの指定チャンネルのコメントをリアルタイムに取得してAIに届けます。
* **超簡単設定**: Botトークンを入力してボタンを押すだけで、招待URLが自動生成されます。
* **完全リアルタイム**: タイムラグなしで瞬時にコメントを拾います。
* **アバター対応**: コメント投稿者のアイコンがreplyテロップに表示されます。

<img width="400" alt="image" src="https://github.com/user-attachments/assets/c8a8585c-52ca-43eb-9cdb-fede4373262a" />

📥 [プラグインをダウンロード](https://github.com/miyumiyu/TeloPon/releases/download/plugins-v1.4/discord_integration.py)  
👉 [詳細な使い方はこちら](../plugins/discord_integration.md)

### 🏢 Slackコメント連携 (`slack_integration.py`)
Slackワークスペースの指定チャンネルのコメントをリアルタイムに取得してAIに届けます。
* **Socket Mode対応**: 企業でも使われる最新の通信方式で、遅延ゼロの完全リアルタイム受信を実現。
* **名前の自動変換**: Slack特有のユーザーIDを、自動的に「発言者の表示名」に変換してAIに読み上げさせます。
* **アバター対応**: コメント投稿者のアイコンがreplyテロップに表示されます。

<img width="400" alt="image" src="https://github.com/user-attachments/assets/1f228c32-3552-453e-aa29-c824da5b6795" />

📥 [プラグインをダウンロード](https://github.com/miyumiyu/TeloPon/releases/download/plugins-v1.4/slack_integration.py)  
👉 [詳細な使い方はこちら](../plugins/slack_integration.md)

---

## 🛠️ プラグインの導入方法（使い方）

1. このページの一番下にある **Assets** から、使いたい機能のファイル（`.py`）をクリックしてダウンロードします。
2. お使いのパソコンにある `TeloPon-XXX` フォルダを開きます。
3. その中にある **`plugins`** フォルダの中に、ダウンロードした `.py` ファイルをポイッと入れます。
4. TeloPonを起動すると、右側の「🔌 拡張機能」の一覧に新しい連携機能が追加されています！

※設定方法の詳細は、各設定画面（⚙️）の中や公式マニュアルをご参照ください。

---

## 📋 リリース件名（GitHub Releases 用）

```
🔌 TeloPon 公式拡張プラグインパック v1.4 (VCI OSC・VOICEVOX・Discord・Slack)
```
