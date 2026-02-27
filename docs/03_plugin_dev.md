# TeloPon: プラグイン開発完全ガイド 🧩

TeloPonの「プラグイン」は、外部の情報をAIに「コッソリ耳打ち」したり、「画像（視覚情報）」を見せたりするための強力な拡張機能です。
「ゲーム内の出来事をAIに伝える」「手動でカンペを出す」「サムネイル画像を見せて感想を言わせる」といったことが、Pythonスクリプトを1つ追加するだけで簡単に実現できます。

このガイドでは、初心者でもコピペで独自のプラグインが作れるように、基本構造から「画像注入」などの高度な実装方法までを徹底解説します。

---

## 📁 1. プラグインの仕組みと保存場所

プラグインは、`TeloPon/plugins/` フォルダ内に保存する単一の Pythonファイル (`.py`) です。
TeloPonを起動すると、このフォルダ内にあるすべてのPythonファイルが自動的に読み込まれ、UIの「🔌 拡張機能」画面に一覧表示されます。

```text
TeloPon/
 └── plugins/
      ├── custom_prompt.py    ← (既存) コメント連携プラグイン
      └── my_plugin.py        ← 🌟 ここに自作のPythonファイルを追加！
```

### 🎯 プラグインの「2つのタイプ」
TeloPonのプラグインには、用途に合わせて2つのタイプ（`PLUGIN_TYPE`）が存在します。

1. **`BACKGROUND`（背景型）**: デフォルトの型。裏で自動的に動き続けるシステム用（時計、コメント取得など）。配信中は設定画面がロックされます。ON/OFFのバッジが付きます。
2. **`TOOL`（ツール型）**: 人間が手動で操作するツール用（手動カンペ送信など）。配信中でも常に操作パネルを開いてボタンを押すことができます。青い「TOOL」バッジが付きます。

---

## 🚀 2. 【コピペ推奨】BACKGROUND型（自動処理）の雛形

まずは、裏で動き続けてAIに自動で情報を送る「BACKGROUND型」の最小構成テンプレートです。
例として「1分ごとにAIに時間を知らせるプラグイン」を作成します。

```python
import time
import threading
import tkinter as tk
from tkinter import ttk
from plugin_manager import BasePlugin

class AutoTimerPlugin(BasePlugin):
    # プラグインのシステム上のID（英数字・他と被らないように）
    PLUGIN_ID = "auto_timer_plugin"
    # UIに表示される名前
    PLUGIN_NAME = "⌚ 自動時報プラグイン"
    # プラグインのタイプ（省略した場合は自動的にBACKGROUNDになります）
    PLUGIN_TYPE = "BACKGROUND"

    def __init__(self):
        super().__init__()
        self.is_running = False
        self.thread = None
        self.plugin_queue = None # AIへのポスト（郵便受け）

    def get_default_settings(self):
        # プラグインの初期設定（デフォルト値）
        return {"enabled": False, "interval_sec": 60}

    def open_settings_ui(self, parent_window):
        """⚙️ 設定画面（UI）を作る部分"""
        if hasattr(self, "settings_win") and self.settings_win is not None and self.settings_win.winfo_exists():
            self.settings_win.lift()
            return
            
        self.settings_win = tk.Toplevel(parent_window)
        self.settings_win.title(f"{self.PLUGIN_NAME} 設定")
        self.settings_win.geometry("300x150")

        settings = self.get_settings()
        var_enabled = tk.BooleanVar(value=settings.get("enabled", False))
        var_interval = tk.IntVar(value=settings.get("interval_sec", 60))

        ttk.Checkbutton(self.settings_win, text="この機能を有効にする", variable=var_enabled).pack(pady=10)
        
        frame = ttk.Frame(self.settings_win)
        frame.pack(pady=5)
        ttk.Label(frame, text="通知する間隔 (秒):").pack(side="left")
        ttk.Entry(frame, textvariable=var_interval, width=5).pack(side="left", padx=5)

        def _save():
            new_settings = {"enabled": var_enabled.get(), "interval_sec": var_interval.get()}
            self.save_settings(new_settings)
            self.settings_win.destroy()

        ttk.Button(self.settings_win, text="保存", command=_save).pack(pady=10)

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

    def stop(self):
        """⏹️ ライブ接続が切断された時に呼ばれる部分"""
        self.is_running = False
        if self.thread and self.thread.is_alive():
            self.thread.join(timeout=1.0)

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
                
                loop_count = 0 # カウンターリセット
```

---

## 🛠️ 3. 【コピペ推奨】TOOL型（手動操作パネル）の雛形

TOOL型は、配信中であっても自由に画面を開いてボタンを押すことができるプラグインです。
「ボタンを押したらAIにカンペを送る」というツールを作ってみましょう。

```python
import tkinter as tk
from tkinter import ttk, messagebox
from plugin_manager import BasePlugin

class ManualButtonPlugin(BasePlugin):
    PLUGIN_ID = "manual_button_plugin"
    PLUGIN_NAME = "🔘 手動カンペボタン"
    PLUGIN_TYPE = "TOOL"  # ★ ここをTOOLにする！

    def __init__(self):
        super().__init__()
        self.plugin_queue = None

    def get_default_settings(self):
        # TOOL型はON/OFFがないため、常にTrueを返しておく
        return {"enabled": True}

    def start(self, prompt_config, plugin_queue):
        """配信開始時にキューを受け取る"""
        self.plugin_queue = plugin_queue

    def stop(self):
        self.plugin_queue = None

    def open_settings_ui(self, parent_window):
        """🖥️ 操作パネルのUI（配信中でも開ける）"""
        self.panel = tk.Toplevel(parent_window)
        self.panel.title("手動カンペパネル")
        self.panel.geometry("300x200")
        self.panel.attributes("-topmost", True) # 常に最前面に表示

        ttk.Label(self.panel, text="AIに指示を送ります", font=("", 12, "bold")).pack(pady=10)

        def _send_cheer():
            if not self.plugin_queue:
                messagebox.showwarning("警告", "ライブ配信を開始してから押してください。")
                return
            
            payload = {
                "type": "text",
                "content": "【ディレクターからの指示】今すぐ配信者を大げさに褒め称えてください！"
            }
            self.plugin_queue.put(payload)
            lbl_status.config(text="✅ 指示を送信しました！", foreground="green")

        ttk.Button(self.panel, text="🎉 配信者を褒めさせる", command=_send_cheer).pack(pady=10)
        
        lbl_status = ttk.Label(self.panel, text="待機中...")
        lbl_status.pack(pady=10)
```

---

## ✉️ 4. AIへのデータ送信フォーマット（テキストと画像）

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

## 🧠 5. 【上級編】起動前にAIの脳内に知識を埋め込む

配信がスタートしてAIが第一声を発する**前**に、プロンプト（AIへの事前指示）にプラグインから追加情報を書き込むことができます。
クラスの中に `get_prompt_addon(self)` というメソッドを定義するだけです。

```python
    def get_prompt_addon(self):
        """配信開始時のシステムプロンプトの末尾に、コッソリ文字を追加します"""
        return """
        【事前知識】
        本日の配信のテーマは「激辛ペヤング完食チャレンジ」です。
        この情報を踏まえて、最初の挨拶から配信者を心配（または煽る）してください。
        """
```
※例えばYouTube連携プラグインでは、ここで「配信タイトル」や「説明文」をAIに事前に読み込ませています。

---

## 🚨 6. プラグイン開発時の注意点とデバッグ

1. **無限ループ内の `time.sleep()` を忘れない**
   `while self.is_running:` のループ内に `time.sleep()` がないと、PCがフリーズします。必ず待機時間を入れてください。
2. **エラー処理（try-except）を入れる**
   ファイル読み込みやAPI通信を行う場合、エラーが起きるとスレッドが強制終了してしまいます。ループ内の処理は `try-except` で囲むのが安全です。
3. **デバッグモードを活用する**
   TeloPonを `-d` オプション（デバッグモード）で起動すると、プラグインからAIにデータが注入された瞬間に、黒い画面に詳細なログや、**AIが受け取った画像がローカルに保存（`debug_latest_image.jpg`）**されるようになります。うまく連携できているかの確認に最適です！