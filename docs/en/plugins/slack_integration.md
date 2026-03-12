# 🏢 Slackコメント連携 (slack_integration.py)

このプラグインを使用すると、会社のSlackワークスペースや、チームのチャンネルに投稿されたコメントを、TeloPonのAIが遅延ゼロで読み上げてくれます！
最新の「Socket Mode」という技術を使っているため、驚くほどリアルタイムに反応します。

設定は大きく分けて **「① プラグインの導入」**、**「② Slack側でのBot作成（2つの鍵の取得）」**、**「③ TeloPon画面での設定」** の3ステップです。少し手順が多いですが、順番通りに進めれば確実に設定できます！

---

## 📥 ステップ1：プラグインの導入と準備

まずはTeloPon本体に拡張機能を追加し、通信に必要なライブラリを準備します。

1. 🔗[ TeloPon 公式拡張プラグインパック v1.0 (Discord & Slack)](https://github.com/miyumiyu/TeloPon/releases/tag/plugins-v1.0)
から、**`slack_integration.py`** をダウンロードします。
2. TeloPon本体のフォルダ内にある **`plugins`** フォルダに、ダウンロードした `slack_integration.py` ファイルを入れます。
3. TeloPonを起動し、メイン画面の「🔌 拡張機能」に **「🏢 Slackコメント連携」** が追加されていれば成功です！

---

## 🛠️ ステップ2：SlackのBotを作成して「2つのトークン」を取得する

AIがSlackのコメントを読むために、専用の「Bot（アプリ）」を作成します。ここでは **2種類のトークン（鍵）** を取得します。

### 1. アプリの作成と名前付け
1. PCのブラウザで [Slack API (Your Apps)](https://api.slack.com/apps) にアクセスし、右上の **「Create New App」** を押します。
2. **「From scratch」** を選び、好きな名前（例：`TeloPon連携Bot`）と導入したいワークスペースを選んで **「Create App」** を押します。
3. **【重要】** 左メニューの **「App Home」** を開き、少し下にある **「Your App’s Presence in Slack」** の「Edit」ボタンを押して、`Display Name` (表示名) と `Default Username` (裏側のID、小文字英数字のみ) を設定して保存します。これをやらないとBotを招待できません！

### 2. Socket ModeをONにして「App-Level トークン」をゲット！
1. 左メニューから **「Socket Mode」** を開きます。
2. **「Enable Socket Mode」** のスイッチを **ON（緑色）** にします。
3. トークンの名前を聞かれるので適当に入力（`telopon-socket`など）して「Generate」を押します。
4. **`xapp-`** から始まる文字列が表示されます。これが **【App-Level トークン】** です。コピーしてメモしておきましょう。

### 3. メッセージを読み取る権限（Scopes）を与える
1. 左メニューから **「OAuth & Permissions」** を開きます。
2. 下にスクロールし、**「Scopes」** 内の **「Bot Token Scopes」** で「Add an OAuth Scope」を押し、以下の5つを追加します。
   * `channels:history` / `groups:history` / `channels:read` / `groups:read` / `users:read`

### 4. リアルタイム受信の設定（Event Subscriptions）
1. 左メニューから **「Event Subscriptions」** を開きます。
2. 一番上の **「Enable Events」** を **ON（緑色）** にします。
3. すぐ下の **「Subscribe to bot events」** を開き、「Add Bot User Event」から以下の2つを追加し、右下の **「Save Changes」** を押します。
   * `message.channels` / `message.groups`

### 5. ワークスペースにインストールして「Bot トークン」をゲット！
1. 左メニューから **「Install App」** を開き、**「Install to Workspace」** を押します。（※権限を変えた場合は「Reinstall」と表示されます）
2. 許可画面が出るので「許可 (Allow)」を押します。
3. **`xoxb-`** から始まる文字列が表示されます。これが **【Bot トークン】** です。コピーしてメモしておきましょう。

### 6. 【超重要】SlackのチャンネルにBotを「招待」する
Slackを開き、**監視させたいチャンネル（#general など）の入力欄で `@TeloPon連携Bot` のようにBot宛にメンション（@）を送信**します。「チャンネルに追加しますか？」と聞かれるので、必ず追加してください。これを忘れるとコメントを読み取れません！

---

## 🖥️ ステップ3：TeloPonのUIで設定・接続する

TeloPonの画面に戻り、「🏢 Slackコメント連携」の設定（⚙️）パネルを開きます。

![Slack連携設定画面](../../../images/discord_integration.png)

1. **トークンの入力**
   * 「App-Level トークン」の欄に、ステップ2で取得した **`xapp-`** から始まる文字を貼り付けます。
   * 「Bot トークン」の欄に、ステップ2で取得した **`xoxb-`** から始まる文字を貼り付けます。
2. **チャンネル一覧の取得**
   * **「🔄 トークン確認 ＆ チャンネル一覧を取得」** ボタンを押します。
   * 成功すると、あなたのSlackにあるチャンネルが一覧で取得されます。
3. **監視するチャンネルを選ぶ**
   * プルダウンから監視したいチャンネルを選びます。（公開チャンネルは💬、鍵付きは🔒アイコンが付きます）
4. **いざ、接続！**
   * 最後に青い **「接続」** ボタンを押して、状態が緑色の「⚡ 状態: 接続中 (Socket Mode稼働中)」になれば準備完了です！

---

## 💡 使い方とワンポイント

設定が完了したら、TeloPonのメイン画面から「ライブ配信に接続（AI起動）」を押すだけです！指定したSlackチャンネルの発言に、AIが素早く反応してくれます。

* **名前の自動変換**: Slack特有のユーザーID（U1234...など）は、TeloPonの裏側で自動的に「発言者の表示名（山田太郎など）」に変換されてAIに届きます。AIはきちんと名前を呼んで返事をしてくれます！
* **プロンプトのカスタマイズ**: 設定画面の下部にある指示テキストを「社内のメンバーからのチャットです！明るく労って！」などに変更すると、AIのキャラクター性をさらに引き出せます。
