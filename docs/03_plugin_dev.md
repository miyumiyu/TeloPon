# TeloPon: プラグイン開発完全ガイド 🧩

TeloPonの「プラグイン」は、外部の情報をAIに「コッソリ耳打ち」したり、「画像（視覚情報）」を見せたりするための強力な拡張機能です。
「ゲーム内の出来事をAIに伝える」「手動でカンペを出す」「DiscordやSlackのコメントを拾う」といったことが、Pythonスクリプトを1つ追加するだけで簡単に実現できます。

このガイドでは、初心者でもコピペで独自のプラグインが作れるように、基本構造から最新の「UI連動」「標準ロガー」の使い方までを徹底解説します。

---

## 📁 1. プラグインの仕組みと保存場所

プラグインは、`TeloPon/plugins/` フォルダ内（または `extended_plugin/` フォルダ内）に保存する単一の Pythonファイル (`.py`) です。
TeloPonを起動すると、これらのフォルダ内にあるすべてのPythonファイルが自動的に読み込まれ、UIの「🔌 拡張機能」画面に一覧表示されます。

```text
TeloPon/
 ├── plugins/
 │    └── custom_prompt.py    ← (既存) コアプラグイン
 └── extended_plugin/
      └── my_plugin.py        ← 🌟 ここに自作のPythonファイルを追加！
```

### 🎯 プラグインの「2つのタイプ」
TeloPonのプラグインには、用途に合わせて2つのタイプ（`PLUGIN_TYPE`）が存在します。

1. **`BACKGROUND`（背景型）**: デフォルトの型。裏で自動的に動き続けるシステム用。配信中は設定画面がロックされます。（ON/OFFのスイッチ型UIにおすすめ）
2. **`TOOL`（ツール型）**: 人間が手動で操作するツール用。配信中でも常に操作パネルを開いてボタンを押すことができます。メイン画面に青い「TOOL」バッジが付きます。

---

## ✨ 2. 【新標準】必須の2大テクニック：Logger と UI連動

最新のTeloPonプラグインでは、プロっぽく仕上げるための**2つの新標準**があります。雛形を見る前に、この2つを覚えておきましょう！

### 📝 ① TeloPon標準ロガー (`logger`)
プラグインの中で `print("テスト")` と書いても良いのですが、TeloPon標準の `logger` を使うと、黒いコンソール画面に**色付きで綺麗に表示**され、エラー原因も分かりやすくなります。

```python
import logger # ★ファイルの先頭でインポート

# 使い方
logger.info(f"[{self.PLUGIN_NAME}] 正常に起動しました！")    # 緑や白の通常ログ
logger.warning(f"[{self.PLUGIN_NAME}] 通信が遅延しています") # 黄色の警告ログ
logger.error(f"[{self.PLUGIN_NAME}] エラーが発生しました！")   # 赤のエラーログ
```

### 🔘 ② 稼働状態のUI連動（接続・切断ボタン）
最近の公式プラグイン（Discord連携など）では、単なるチェックボックスではなく、**「接続 / 切断」ボタン**を使ってリアルタイムに状態を切り替えるのが主流です。
これを行うと、TeloPonメイン画面のバッジの色も連動して光るようになり、ユーザー体験（UX）が劇的に向上します。

---

## 🚀 3. 【コピペ推奨】本格派 BACKGROUND型 の雛形

それでは、裏で動き続けてAIに情報を送る「BACKGROUND型」の最新テンプレートです。
例として、標準ロガーとUI連動を組み込んだ**「1分ごとにAIに時間を知らせるプラグイン」**を作成します。これをコピペして名前を変えれば、すぐに自作プラグインが完成します！

```python
import time
import threading
import tkinter as tk
from tkinter import ttk, messagebox
from plugin_manager import BasePlugin
import logger # ★ TeloPon標準ロガー

class AutoTimerPlugin(BasePlugin):
    PLUGIN_ID = "auto_timer_plugin"
    PLUGIN_NAME = "⌚ 自動時報プラグイン"
    PLUGIN_TYPE = "BACKGROUND"

    def __init__(self):
        super().__init__()
        self.plugin_queue = None
        self.is_running = False
        self.is_connected = False # ★ UI・システム連動のための接続フラグ
        self.thread = None

    def get_default_settings(self):
        # enabled が True の時だけ起動します
        return {"enabled": False, "interval_sec": 60}

    # ==========================================
    # ⚙️ 設定UIの構築（モダンな接続ボタン搭載）
    # ==========================================
    def open_settings_ui(self, parent_window):
        if hasattr(self, "panel") and self.panel is not None and self.panel.winfo_exists():
            self.panel.lift()
            return
            
        self.panel = tk.Toplevel(parent_window)
        self.panel.title(f"{self.PLUGIN_NAME} 設定")
        self.panel.geometry("350x250")
        self.panel.attributes("-topmost", True)

        settings = self.get_settings()
        self.is_connected = settings.get("enabled", False)

        main_f = ttk.Frame(self.panel, padding=15)
        main_f.pack(fill=tk.BOTH, expand=True)

        # 1. 状態表示ラベル
        self.lbl_status = ttk.Label(main_f, text="🔌 状態: 停止中", font=("", 11, "bold"), foreground="gray")
        self.lbl_status.pack(anchor="w", pady=(0, 15))

        # 2. 設定項目
        frame = ttk.Frame(main_f)
        frame.pack(fill=tk.X, pady=10)
        ttk.Label(frame, text="通知する間隔 (秒):").pack(side="left")
        self.ent_interval = ttk.Entry(frame, width=10)
        self.ent_interval.pack(side="left", padx=5)
        self.ent_interval.insert(0, str(settings.get("interval_sec", 60)))

        # 3. 接続/切断ボタン
        self.btn_connect = tk.Button(main_f, text="起動する", bg="#007bff", fg="white", font=("", 10, "bold"), command=self._toggle_connection)
        self.btn_connect.pack(fill=tk.X, pady=10)

        # 4. 保存して閉じるボタン
        tk.Button(main_f, text="保存して閉じる", bg="#6c757d", fg="white", command=self._save_and_close).pack(fill=tk.X, pady=(10, 0))

        self._update_ui_state() # UIの初期表示を更新

    def _update_ui_state(self):
        """接続状態に合わせてUI（文字とボタンの色・ロック状態）を切り替える"""
        if not hasattr(self, 'btn_connect') or not self.btn_connect.winfo_exists(): return
        
        if self.is_connected:
            self.lbl_status.config(text="⚡ 状態: 稼働中 (時間を監視しています)", foreground="#28a745")
            self.btn_connect.config(text="停止する", bg="#dc3545")
            self.ent_interval.config(state="disabled") # 稼働中は設定を変えられないようにロック
        else:
            self.lbl_status.config(text="🔌 状態: 停止中", foreground="gray")
            self.btn_connect.config(text="起動する", bg="#007bff")
            self.ent_interval.config(state="normal")

    def _toggle_connection(self):
        """接続ボタンが押された時の処理"""
        if self.is_connected:
            self.is_connected = False
            self._update_ui_state()
            self._save_settings_from_ui()
            logger.info(f"[{self.PLUGIN_NAME}] ⏹️ 機能を手動で停止しました。")
        else:
            try:
                int(self.ent_interval.get()) # 数字かチェック
            except ValueError:
                messagebox.showwarning("警告", "秒数は数字で入力してください。")
                return

            self.is_connected = True
            self._update_ui_state()
            self._save_settings_from_ui()
            logger.info(f"[{self.PLUGIN_NAME}] ▶️ 機能を有効にしました！配信開始時に稼働します。")

    def _save_settings_from_ui(self):
        settings = self.get_settings()
        settings["enabled"] = self.is_connected
        settings["interval_sec"] = int(self.ent_interval.get())
        self.save_settings(settings)

    def _save_and_close(self):
        self._save_settings_from_ui()
        self.panel.destroy()

    # ==========================================
    # 🧠 AI連携 & バックグラウンド処理
    # ==========================================
    def start(self, prompt_config, plugin_queue):
        """▶️ ライブ接続が開始された時に呼ばれる部分"""
        settings = self.get_settings()
        if not settings.get("enabled"): 
            return

        if self.is_running: return
        
        self.is_running = True
        self.plugin_queue = plugin_queue # ★AIへのパイプを受け取る
        
        # TeloPon本体を止めないよう、別スレッドで処理を走らせる
        self.thread = threading.Thread(target=self._background_loop, args=(prompt_config,), daemon=True)
        self.thread.start()
        logger.info(f"[{self.PLUGIN_NAME}] ⏳ 裏での時間監視ループを開始しました。")

    def stop(self):
        """⏹️ ライブ接続が切断された時に呼ばれる部分"""
        self.is_running = False
        self.plugin_queue = None

    def _background_loop(self, prompt_config):
        """🔄 裏でずっと動き続けるメイン処理"""
        settings = self.get_settings()
        interval = int(settings.get("interval_sec", 60))
        ai_name = prompt_config.get("ai_name", "AI")
        
        loop_count = 0
        while self.is_running:
            time.sleep(1.0) # CPU負荷を下げるための必須スリープ
            loop_count += 1
            
            if loop_count >= interval:
                current_time = time.strftime("%H時%M分")
                
                # ★ AIへ送るテキストの作成（辞書型）
                payload = {
                    "type": "text",
                    "content": f"【システム通知】現在時刻は {current_time} です。{ai_name}、時間を知らせて話題を切り替えてください。"
                }
                
                # 📮 AIの脳内に注入！
                if self.plugin_queue:
                    self.plugin_queue.put(payload)
                    logger.info(f"[{self.PLUGIN_NAME}] ⚡ AIに時刻({current_time})を送信しました！")
                
                loop_count = 0 # カウンターリセット
```

---

## 🛠️ 4. 【コピペ推奨】TOOL型（手動操作パネル）の雛形

TOOL型は、配信中であっても自由に画面を開いてボタンを押すことができるプラグインです。
「ボタンを押したらAIにカンペを送る」というツールを作ってみましょう。

```python
import tkinter as tk
from tkinter import ttk, messagebox
from plugin_manager import BasePlugin
import logger

class ManualButtonPlugin(BasePlugin):
    PLUGIN_ID = "manual_button_plugin"
    PLUGIN_NAME = "🔘 手動カンペボタン"
    PLUGIN_TYPE = "TOOL"  # ★ ここをTOOLにする！

    def __init__(self):
        super().__init__()
        self.plugin_queue = None

    def get_default_settings(self):
        # TOOL型はシステム連動のON/OFFがないため、常にTrueを返しておくのが基本です
        return {"enabled": True}

    def start(self, prompt_config, plugin_queue):
        """配信開始時にAIへの送信パイプを受け取る"""
        self.plugin_queue = plugin_queue
        logger.info(f"[{self.PLUGIN_NAME}] 準備完了。いつでもカンペを送れます。")

    def stop(self):
        self.plugin_queue = None

    def open_settings_ui(self, parent_window):
        """🖥️ 操作パネルのUI（配信中でも開ける）"""
        if hasattr(self, "panel") and self.panel is not None and self.panel.winfo_exists():
            self.panel.lift()
            return

        self.panel = tk.Toplevel(parent_window)
        self.panel.title(self.PLUGIN_NAME)
        self.panel.geometry("300x200")
        self.panel.attributes("-topmost", True) # 常に最前面に表示

        ttk.Label(self.panel, text="AIに指示を送ります", font=("", 12, "bold")).pack(pady=10)

        def _send_cheer():
            if not self.plugin_queue:
                messagebox.showwarning("警告", "TeloPonのライブ配信を開始してから押してください。")
                return
            
            payload = {
                "type": "text",
                "content": "【ディレクターからの指示】今すぐ配信者を大げさに褒め称えてください！"
            }
            self.plugin_queue.put(payload)
            
            lbl_status.config(text="✅ 指示を送信しました！", foreground="green")
            logger.info(f"[{self.PLUGIN_NAME}] 手動カンペを送信しました。")

        ttk.Button(self.panel, text="🎉 配信者を褒めさせる", command=_send_cheer).pack(pady=10)
        
        lbl_status = ttk.Label(self.panel, text="待機中...")
        lbl_status.pack(pady=10)
```

---

## ✉️ 5. AIへのデータ送信フォーマット（テキストと画像）

AIへの情報注入は、`self.plugin_queue.put()` メソッドを使って行います。送信するデータの種類によって「辞書（Dictionary）」の形を変えます。

### 📝 パターンA: テキスト（カンペ・指示）を注入する
```python
payload = {
    "type": "text",
    "content": "【カンペ】ここ笑うところだよ！"
}
self.plugin_queue.put(payload)
```
> **💡 コツ**: 単なるデータ（「りんご」など）を送るのではなく、「りんごの画像です。これについて感想を言って！」のように**AIへの具体的な指示（行動のフック）を含める**と、確実にリアクションしてくれます。

### 📸 パターンB: 画像（視覚情報）を注入する
AIに画像を見せる場合は、画像を**JPEGやPNGのバイナリデータ（Bytes）**に変換して送ります。※画像の処理には `Pillow` ライブラリを使用するのが一般的です。

```python
import io
from PIL import Image

# 1. 画像を開いてリサイズする（大きすぎると重いため、最大1024px推奨）
img = Image.open("sample.jpg")
if img.mode != "RGB":
    img = img.convert("RGB")
img.thumbnail((1024, 1024))

# 2. バイナリデータ(Bytes)に変換する
img_byte_arr = io.BytesIO()
img.save(img_byte_arr, format='JPEG', quality=85)
img_bytes = img_byte_arr.getvalue()

# 3. ペイロードを作って送信！
payload = {
    "type": "image",
    "data": img_bytes,
    "mime_type": "image/jpeg"
}
self.plugin_queue.put(payload)
```
> **⚠️ 画像注入の注意点**: 画像だけを投げつけてもAIは「何のために見せられたのか」分かりません。画像を送った直後に、**「今見せた画像についてツッコミを入れて！」というテキスト（パターンA）も連続して送信する**のが完璧なテクニックです。

---

## 🧠 6. 【上級編】起動前にAIの脳内に知識を埋め込む

配信がスタートしてAIが第一声を発する**前**に、プロンプト（AIへの事前指示）にプラグインから追加情報を書き込むことができます。
クラスの中に `get_prompt_addon(self)` というメソッドを定義するだけです。

```python
    def get_prompt_addon(self):
        """配信開始時のシステムプロンプトの末尾に、コッソリ文字を追加します"""
        # ※設定画面でONになっているかチェックしてから返すのが丁寧です
        settings = self.get_settings()
        if not settings.get("enabled"): return ""

        return """
        【事前知識】
        本日の配信のテーマは「激辛ペヤング完食チャレンジ」です。
        この情報を踏まえて、最初の挨拶から配信者を心配（または煽る）してください。
        """
```

---

## 🚨 7. プラグイン開発時の注意点とデバッグ

1. **無限ループ内の `time.sleep()` を忘れない**
   `while self.is_running:` のループ内に `time.sleep()` がないと、PCがフリーズします。必ず待機時間を入れてください。
2. **エラー処理（try-except）を入れる**
   ファイル読み込みやAPI通信を行う場合、エラーが起きるとスレッドが強制終了してしまいます。ループ内の処理は `try-except` で囲み、エラー時は `logger.error(e)` で出力するのが安全です。
3. **デバッグモードを活用する**
   TeloPonを `-d` オプション（デバッグモード）で起動すると、プラグインからAIにデータが注入された瞬間に、黒い画面に詳細なログや、**AIが受け取った画像がローカルに保存（`debug_latest_image.jpg`）**されるようになります。うまく連携できているかの確認に最適です！