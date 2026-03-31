# TeloPon: プラグイン開発完全ガイド 🧩

TeloPonのプラグインは、大きく分けて **2種類** あります。

| 種類 | 説明 | 登録方法 |
|------|------|----------|
| **UIパネルプラグイン** | TeloPonの「拡張機能」画面に表示。AIへの情報注入や設定パネルを提供する | `plugins/` フォルダに置くだけで自動登録 |
| **CMDコマンドプラグイン** | AIが `[CMD:XXX]` を発言したときに呼ばれる処理を担当する | `plugins/` フォルダに置くだけで自動登録 |

どちらも同じ `BasePlugin` クラスを継承して作ります。
両方の機能を **1つのクラスに組み合わせる** こともできます。

---

## 1. 共通の基盤：BasePlugin クラス

すべてのプラグインは `plugin_manager.py` の `BasePlugin` を継承して作ります。

```python
from plugin_manager import BasePlugin
```

`BasePlugin` には、UIパネル用とCMD用の **2セットのフィールド・メソッド** があります。
使いたい種類のものだけを設定・実装すればOKです。

### クラス変数一覧

```python
class BasePlugin:
    # ===== UIパネルプラグイン用 =====
    PLUGIN_ID   = ""           # ← 空のままだとUIパネルに登録されない
    PLUGIN_NAME = "Base Plugin"
    PLUGIN_TYPE = "BACKGROUND" # "BACKGROUND" / "TOOL"

    # ===== CMDコマンドプラグイン用 =====
    IDENTIFIER      = ""       # ← [CMD:XXX] の XXX。空のままだとCMDに登録されない
    REQUIRED_CONFIG: list = [] # setup() で事前チェックするconfigキー名のリスト
```

`PLUGIN_ID` を設定すると → UIパネルプラグインとして機能する
`IDENTIFIER` を設定すると → CMDコマンドプラグインとして機能する
**両方設定すると → 両方の機能を持つハイブリッドプラグインになる**

---

## 2. PLUGIN_TYPE の2種類

UIパネルプラグインを作る場合、`PLUGIN_TYPE` を以下のどちらかに設定します。

| PLUGIN_TYPE | 説明 |
|------------|------|
| `"BACKGROUND"` | 配信中は設定画面がロックされる。裏で自動的に動くシステム向け |
| `"TOOL"` | 配信中でもいつでも操作パネルを開いて操作できる。手動ツール向け |

---

## 3. バッジ表示の仕組み

拡張機能パネルの各プラグインには **TOOL / ON / OFF** のバッジが表示されます。
バッジの色（アクティブ=青/緑 か グレーか）は、以下の優先順位で決まります。

```
優先度1: is_connected 属性が True  → アクティブ（青/緑）
優先度2: is_running   属性が True  → アクティブ（青/緑）
フォールバック: 上記どちらも属性として存在しない → enabled 設定値を使用
```

### 状態の組み合わせと結果（全パターン）

| `is_connected` | `is_running` | `enabled` | バッジ |
|---|---|---|---|
| True（定義あり） | ─ | ─ | アクティブ（青/緑） |
| False（定義あり） | ─ | ─ | グレー |
| 未定義 | True | ─ | アクティブ（青/緑） |
| 未定義 | False | ─ | グレー |
| 未定義 | 未定義 | True | アクティブ（青/緑） |
| 未定義 | 未定義 | False | グレー |

> **`is_running` はライブセッションのイベントに連動します。**
> TeloPonがGemini Liveへの接続・切断を行うと、全プラグインの `start()` / `stop()` が呼ばれます。
> `start()` 内で `self.is_running = True`、`stop()` 内で `self.is_running = False` にするのが典型的な使い方です。

### パターン別の動作（用途の早見表）

| プラグインの実装 | バッジがアクティブになる条件 | 典型的な用途 |
|---|---|---|
| `is_connected` 定義あり・True固定 | 常にアクティブ | 外部接続不要だが常時アクティブにしたいTOOL |
| `is_connected` 定義あり・可変 | `is_connected = True` のとき | YouTube・Twitch など外部サービス接続型 |
| `is_running` 定義あり（`is_connected` なし） | `is_running = True` のとき | バックグラウンド処理型 |
| どちらも未定義 | `enabled = True` のとき | シンプルなカスタムプラグイン |

### 重要：`is_connected` を定義するかどうか

**外部サービスに接続するプラグイン（YouTube・Twitch等）** は `is_connected` を定義し、
接続済みのときだけ `True` にします。未接続でもバッジが光るのは正しくないためです。

```python
# 外部サービス接続型：is_connected で制御する
class YouTubePlugin(BasePlugin):
    def __init__(self):
        super().__init__()
        self.is_connected = False  # 接続完了時に True にする
        self.is_running   = False  # ライブ開始時に True にする
```

**シンプルなカスタムプラグイン** は `is_connected` / `is_running` を定義しないことで、
`enabled = True` だけでバッジがアクティブになります。

```python
# シンプルなカスタムプラグイン：enabled で制御する（is_connected を定義しない）
class MySimplePlugin(BasePlugin):
    PLUGIN_ID   = "my_plugin"
    PLUGIN_TYPE = "TOOL"

    def get_default_settings(self):
        return {"enabled": True}  # これだけでライブ前からバッジが光る

    # is_connected / is_running は定義しない ← ポイント
```

---

## 4. UIパネルプラグインの作り方

### ファイルの置き場所

```
TeloPon/
 └── plugins/
      └── my_plugin.py   ← ここに置くだけで自動的にUIに表示される
```

`PLUGIN_ID` を設定していれば、アプリ起動時に `plugins/` フォルダを自動スキャンして登録されます。

### 最小構成の例

```python
import tkinter as tk
from tkinter import ttk
from plugin_manager import BasePlugin
import logger

class MyPlugin(BasePlugin):
    PLUGIN_ID   = "my_plugin"      # ← ユニークなID（ファイル名と合わせると分かりやすい）
    PLUGIN_NAME = "🔧 サンプルプラグイン"
    PLUGIN_TYPE = "BACKGROUND"

    def get_default_settings(self):
        """設定の初期値を返す"""
        return {"enabled": False, "message": "こんにちは"}

    def open_settings_ui(self, parent_window):
        """⚙️ 設定ボタンを押したときに開くウィンドウ"""
        panel = tk.Toplevel(parent_window)
        panel.title(self.PLUGIN_NAME)
        panel.geometry("300x150")
        panel.attributes("-topmost", True)

        settings = self.get_settings()

        ttk.Label(panel, text="メッセージ:").pack(pady=5)
        ent = ttk.Entry(panel, width=30)
        ent.insert(0, settings.get("message", ""))
        ent.pack()

        def save():
            s = self.get_settings()
            s["message"] = ent.get()
            self.save_settings(s)
            panel.destroy()

        ttk.Button(panel, text="保存", command=save).pack(pady=10)

    def start(self, prompt_config, plugin_queue):
        """▶️ ライブ配信開始時に呼ばれる"""
        logger.info(f"[{self.PLUGIN_NAME}] 起動しました")

    def stop(self):
        """⏹️ ライブ配信終了時に呼ばれる"""
        logger.info(f"[{self.PLUGIN_NAME}] 停止しました")
```

### 設定ファイルについて

設定は `plugins/my_plugin.json` に自動で保存されます。手動でファイルを操作する必要はありません。

```python
# 読み込み（get_default_settings の値がベースになる）
settings = self.get_settings()
value = settings.get("my_key", "デフォルト値")

# 保存
settings = self.get_settings()
settings["my_key"] = "新しい値"
self.save_settings(settings)
```

### get_display_name のオーバーライド（多言語対応）

UI に表示されるプラグイン名を動的に変えたい場合（多言語対応など）は `get_display_name()` をオーバーライドします。

```python
def get_display_name(self) -> str:
    import i18n
    return i18n.t("my_plugin_name")  # 言語設定に応じた名前を返す
```

---

## 5. AIへの情報注入（UIパネルプラグインの主な役割）

### 方法① 配信開始前にプロンプトへ追記する

```python
def get_prompt_addon(self):
    """配信開始直前に呼ばれる。AIのシステムプロンプトに追記する文字列を返す。"""
    settings = self.get_settings()
    if not settings.get("enabled"):
        return ""
    return """
    【事前情報】
    本日は視聴者100人達成記念配信です。
    AIはこの情報を踏まえてお祝いのコメントを冒頭に入れてください。
    """
```

有効になっている全プラグインの `get_prompt_addon()` は自動的に結合されてAIに渡されます。
どのプラグインからの情報かわかるよう、ヘッダーが自動付与されます。

### 方法② 配信中にリアルタイムでAIへテキストを送る

```python
def start(self, prompt_config, plugin_queue):
    self.plugin_queue = plugin_queue  # キューを保持しておく
    # ... スレッド起動など

def _some_event_happened(self):
    # AIにテキストを注入（専用メソッドを使うと便利）
    self.send_text(self.plugin_queue, "【通知】視聴者が増えました！触れてください。")
    # または直接 put しても同じ
    # self.plugin_queue.put({"type": "text", "content": "..."})
```

### 方法③ 配信中にリアルタイムでAIへ画像を送る

```python
import io
from PIL import Image

def _send_screenshot(self):
    img = Image.open("screenshot.png")
    if img.mode != "RGB":
        img = img.convert("RGB")
    img.thumbnail((1024, 1024))

    buf = io.BytesIO()
    img.save(buf, format="JPEG", quality=85)

    # 専用メソッドを使うと便利（mime_type 省略時は "image/jpeg"）
    self.send_image(self.plugin_queue, buf.getvalue())
    # その後、何の画像か指示も送るのがポイント！
    self.send_text(self.plugin_queue, "今見せた画像についてコメントしてください。")
```

### start() の prompt_config について

`start(self, prompt_config, plugin_queue)` の `prompt_config` には、現在選択中のプロンプト設定が渡されます。

```python
def start(self, prompt_config, plugin_queue):
    ai_name  = prompt_config.get("ai_name", "AI")    # AI のキャラクター名
    language = prompt_config.get("language", "ja")   # 言語設定
```

---

## 6. 本格的なBACKGROUND型プラグインの雛形

裏で動き続け、定期的にAIに情報を送るプラグインの完全テンプレートです。

```python
import time
import threading
import tkinter as tk
from tkinter import ttk, messagebox
from plugin_manager import BasePlugin
import logger


class AutoTimerPlugin(BasePlugin):
    PLUGIN_ID   = "auto_timer_plugin"
    PLUGIN_NAME = "⌚ 自動時報プラグイン"
    PLUGIN_TYPE = "BACKGROUND"

    def __init__(self):
        super().__init__()
        self.plugin_queue = None
        self.is_running   = False
        self.is_connected = False  # UIと連動する接続状態フラグ（バッジ表示にも使われる）
        self.thread       = None

    def get_default_settings(self):
        return {"enabled": False, "interval_sec": 60}

    # ==========================================
    # 設定UI
    # ==========================================
    def open_settings_ui(self, parent_window):
        # すでに開いていたら前面に出す
        if hasattr(self, "panel") and self.panel is not None and self.panel.winfo_exists():
            self.panel.lift()
            return

        self.panel = tk.Toplevel(parent_window)
        self.panel.title(f"{self.PLUGIN_NAME} 設定")
        self.panel.geometry("350x220")
        self.panel.attributes("-topmost", True)

        settings = self.get_settings()
        self.is_connected = settings.get("enabled", False)

        main_f = ttk.Frame(self.panel, padding=15)
        main_f.pack(fill=tk.BOTH, expand=True)

        # 状態ラベル
        self.lbl_status = ttk.Label(main_f, text="", font=("", 11, "bold"))
        self.lbl_status.pack(anchor="w", pady=(0, 10))

        # 設定入力
        row = ttk.Frame(main_f)
        row.pack(fill=tk.X, pady=5)
        ttk.Label(row, text="通知する間隔（秒）:").pack(side="left")
        self.ent_interval = ttk.Entry(row, width=8)
        self.ent_interval.pack(side="left", padx=5)
        self.ent_interval.insert(0, str(settings.get("interval_sec", 60)))

        # 接続/切断ボタン
        self.btn_connect = tk.Button(
            main_f, font=("", 10, "bold"),
            command=self._toggle_connection
        )
        self.btn_connect.pack(fill=tk.X, pady=8)

        # 保存して閉じるボタン
        tk.Button(
            main_f, text="保存して閉じる",
            bg="#6c757d", fg="white",
            command=self._save_and_close
        ).pack(fill=tk.X)

        self._update_ui()  # UIの初期状態を反映

    def _update_ui(self):
        """接続状態に合わせてUIの見た目を切り替える"""
        if not hasattr(self, "btn_connect") or not self.btn_connect.winfo_exists():
            return
        if self.is_connected:
            self.lbl_status.config(text="⚡ 稼働中", foreground="#28a745")
            self.btn_connect.config(text="停止する", bg="#dc3545", fg="white")
            self.ent_interval.config(state="disabled")  # 稼働中はロック
        else:
            self.lbl_status.config(text="🔌 停止中", foreground="gray")
            self.btn_connect.config(text="起動する", bg="#007bff", fg="white")
            self.ent_interval.config(state="normal")

    def _toggle_connection(self):
        if self.is_connected:
            self.is_connected = False
            self._update_ui()
            self._save_from_ui()
            logger.info(f"[{self.PLUGIN_NAME}] 停止しました")
        else:
            try:
                int(self.ent_interval.get())
            except ValueError:
                messagebox.showwarning("エラー", "秒数は数字で入力してください")
                return
            self.is_connected = True
            self._update_ui()
            self._save_from_ui()
            logger.info(f"[{self.PLUGIN_NAME}] 有効にしました")

    def _save_from_ui(self):
        s = self.get_settings()          # ← 現在の設定を読み込む
        s["enabled"]      = self.is_connected
        s["interval_sec"] = int(self.ent_interval.get())
        self.save_settings(s)

    def _save_and_close(self):
        self._save_from_ui()
        self.panel.destroy()

    # ==========================================
    # ライフサイクル（ライブ配信の開始/停止）
    # ==========================================
    def start(self, prompt_config, plugin_queue):
        settings = self.get_settings()
        if not settings.get("enabled"):
            return
        if self.is_running:
            return

        self.is_running   = True
        self.plugin_queue = plugin_queue
        self.thread = threading.Thread(
            target=self._loop,
            args=(prompt_config,),
            daemon=True
        )
        self.thread.start()
        logger.info(f"[{self.PLUGIN_NAME}] バックグラウンドループ開始")

    def stop(self):
        self.is_running   = False
        self.plugin_queue = None

    # ==========================================
    # バックグラウンド処理（別スレッドで動く）
    # ==========================================
    def _loop(self, prompt_config):
        settings     = self.get_settings()
        interval     = int(settings.get("interval_sec", 60))
        ai_name      = prompt_config.get("ai_name", "AI")
        elapsed      = 0

        while self.is_running:
            time.sleep(1.0)  # ← 必須！ないとCPUが100%になる
            elapsed += 1

            if elapsed >= interval:
                current_time = time.strftime("%H時%M分")
                self.send_text(
                    self.plugin_queue,
                    f"【システム通知】現在時刻は {current_time} です。{ai_name}、時間を知らせてください。"
                )
                logger.info(f"[{self.PLUGIN_NAME}] 時刻をAIに送信: {current_time}")
                elapsed = 0
```

---

## 7. TOOL型プラグインの雛形

配信中でも操作できる手動ツールです。`PLUGIN_TYPE = "TOOL"` にするだけで配信中もパネルが操作可能になります。

### シンプルなTOOL型（enabled のみでバッジ制御）

`is_connected` / `is_running` を定義しないシンプルな構成です。
`enabled = True` のままバッジがアクティブになるため、ライブ前から青く光ります。

```python
import tkinter as tk
from tkinter import ttk, messagebox
from plugin_manager import BasePlugin
import logger


class ManualCuePlugin(BasePlugin):
    PLUGIN_ID   = "manual_cue_plugin"
    PLUGIN_NAME = "🎬 手動カンペ送信"
    PLUGIN_TYPE = "TOOL"  # ← これがポイント

    def __init__(self):
        super().__init__()
        self.plugin_queue = None
        # ★ is_connected / is_running を定義しない
        #   → get_default_settings の enabled 値でバッジが制御される

    def get_default_settings(self):
        return {"enabled": True}  # True にするとライブ前からバッジが青く光る

    def start(self, prompt_config, plugin_queue):
        self.plugin_queue = plugin_queue

    def stop(self):
        self.plugin_queue = None

    def open_settings_ui(self, parent_window):
        if hasattr(self, "panel") and self.panel is not None and self.panel.winfo_exists():
            self.panel.lift()
            return

        self.panel = tk.Toplevel(parent_window)
        self.panel.title(self.PLUGIN_NAME)
        self.panel.geometry("320x200")
        self.panel.attributes("-topmost", True)

        ttk.Label(self.panel, text="AIへ送るメッセージ", font=("", 11)).pack(pady=10)

        ent = ttk.Entry(self.panel, width=35)
        ent.pack(pady=5)
        ent.insert(0, "今すぐ笑いを取りに行ってください！")

        lbl = ttk.Label(self.panel, text="")
        lbl.pack(pady=5)

        def send():
            if not self.plugin_queue:
                messagebox.showwarning("警告", "配信を開始してから押してください")
                return
            msg = ent.get().strip()
            if not msg:
                return
            self.send_text(self.plugin_queue, f"【ディレクター指示】{msg}")
            lbl.config(text="✅ 送信しました！", foreground="green")
            logger.info(f"[{self.PLUGIN_NAME}] 手動カンペ送信: {msg}")

        ttk.Button(self.panel, text="📨 AIに送る", command=send).pack(pady=10)
```

### is_connected=True 固定パターン（常時アクティブ型）

外部サービスへの接続は不要だが、設定の `enabled` に関わらず常にバッジをアクティブにしたい場合は
`is_connected = True` を固定で設定します。

```python
class AlwaysActiveToolPlugin(BasePlugin):
    PLUGIN_ID   = "always_active_plugin"
    PLUGIN_NAME = "🔧 常時アクティブプラグイン"
    PLUGIN_TYPE = "TOOL"

    def __init__(self):
        super().__init__()
        self.is_connected = True  # 固定でTrue → 常にバッジがアクティブ
        self.plugin_queue = None

    def get_default_settings(self):
        return {"enabled": True}

    def start(self, prompt_config, plugin_queue):
        self.plugin_queue = plugin_queue

    def stop(self):
        self.plugin_queue = None
```

### is_connected を使うTOOL型（外部サービス接続型）

YouTube・Twitch 等の外部サービスに接続するプラグインのパターンです。
`is_connected = True` になったときだけバッジが青くなります。

```python
class ExternalServicePlugin(BasePlugin):
    PLUGIN_ID   = "my_service_plugin"
    PLUGIN_NAME = "🔗 外部サービス連携"
    PLUGIN_TYPE = "TOOL"

    def __init__(self):
        super().__init__()
        self.plugin_queue = None
        self.is_connected = False  # ← 接続完了時に True にする（バッジ表示に使われる）
        self.is_running   = False  # ← ライブ中に True にする

    def get_default_settings(self):
        return {"enabled": True, "url": ""}

    def _connect(self, url):
        # ... 接続処理 ...
        self.is_connected = True   # ← 接続成功でバッジが青くなる

    def _disconnect(self):
        self.is_connected = False  # ← 切断でバッジがグレーになる

    def start(self, prompt_config, plugin_queue):
        if not self.is_connected:
            return  # 未接続なら起動しない
        self.is_running   = True
        self.plugin_queue = plugin_queue

    def stop(self):
        self.is_running   = False
        self.plugin_queue = None
```

---

## 8. CMDコマンドプラグインの作り方

AIが発言の中に `[CMD:XXX] 引数` という形式を書いたときに処理を実行するプラグインです。

### 仕組み

```
AIの発言:  「それでは画像を表示します [CMD:IMG] create 夕焼けの風景」
              ↓
TeloPonが [CMD:IMG] を検出
              ↓
ImgPlugin の handle("create 夕焼けの風景") が呼ばれる
```

### 最小構成の例

```python
from plugin_manager import BasePlugin
import logger


class MyCommandPlugin(BasePlugin):
    IDENTIFIER = "MYCMD"   # ← [CMD:MYCMD] に反応する。PLUGIN_IDは設定しない
    REQUIRED_CONFIG = []

    def handle(self, value: str):
        """[CMD:MYCMD] value の部分が value として渡される"""
        logger.info(f"[CMD:MYCMD] 受信: {value}")
        # ここに処理を書く


plugin = MyCommandPlugin()
```

### 自動登録の仕組み

CMDプラグインは **`plugins/` フォルダに `.py` ファイルを置くだけで自動登録されます**。
`IDENTIFIER` を設定していれば、アプリ起動時に自動スキャンされます。

```
TeloPon/
 └── plugins/
      └── my_command_plugin.py   ← ここに置くだけで [CMD:MYCMD] が有効になる
```

### REQUIRED_CONFIG の使い方

APIキーなど、設定がないと動けない場合は `REQUIRED_CONFIG` に設定キーを列挙すると、
未設定の場合に自動でスキップされ、エラーを防げます。

```python
class WeatherPlugin(BasePlugin):
    IDENTIFIER      = "WEATHER"
    REQUIRED_CONFIG = ["weather_api_key"]  # config.weather_api_key が必須

    def handle(self, value: str):
        import config
        api_key = config.app_config["weather_api_key"]
        # ...
```

---

## 9. UIパネルとCMDを兼ねたハイブリッドプラグイン

`PLUGIN_ID` と `IDENTIFIER` を両方設定すると、UIパネルとCMDコマンドの両方の機能を持てます。

```python
from plugin_manager import BasePlugin
import logger


class HybridPlugin(BasePlugin):
    # UIパネルとして登録される
    PLUGIN_ID   = "hybrid_plugin"
    PLUGIN_NAME = "🔀 ハイブリッドプラグイン"
    PLUGIN_TYPE = "TOOL"

    # CMDコマンドとしても登録される
    IDENTIFIER = "HYBRID"

    def __init__(self):
        super().__init__()
        self.plugin_queue = None

    # ----- UIパネル側 -----
    def start(self, prompt_config, plugin_queue):
        self.plugin_queue = plugin_queue

    def stop(self):
        self.plugin_queue = None

    def open_settings_ui(self, parent_window):
        # ... 設定パネルUI

    # ----- CMDコマンド側 -----
    def handle(self, value: str):
        """[CMD:HYBRID] を受け取ったときの処理"""
        logger.info(f"[CMD:HYBRID] 受信: {value}")
        if self.plugin_queue:
            self.send_text(self.plugin_queue, f"ハイブリッド処理: {value}")
```

ハイブリッドプラグインも `plugins/` フォルダに置くだけで自動登録されます。

---

## 10. 実装済みプラグインの例：img_plugin.py

実際に使われているCMDプラグインの例です（`plugins/img_plugin.py`）。

```
[CMD:IMG] create 夕焼けの猫   → Gemini AIで画像を生成して表示
[CMD:IMG] title               → TITLE.png を大きく表示
[CMD:IMG] last                → 前回生成した画像を再表示
[CMD:IMG] hide                → 画像を非表示
[CMD:IMG] big                 → 画像を大きく表示
[CMD:IMG] small               → 画像を縮小表示
```

`ImgPlugin` はCMD専用なので `PLUGIN_ID` は持たず（UIパネルには出ない）、
`IDENTIFIER = "IMG"` のみ設定しています。

---

## 11. よく使うメソッド・便利機能まとめ

### send_text / send_image（BasePlugin に標準搭載）

```python
# テキストをAIに送る
self.send_text(self.plugin_queue, "AIへのメッセージ")

# 画像をAIに送る
# send_image(queue, image_bytes, mime_type="image/jpeg")
#   queue       : self.plugin_queue を渡す
#   image_bytes : bytes形式の画像データ
#   mime_type   : 省略可。省略時は "image/jpeg"（位置引数・キーワード引数どちらでも可）
self.send_image(self.plugin_queue, img_bytes, mime_type="image/jpeg")
```

### logger（TeloPon標準ログ出力）

```python
import logger

logger.info("通常のログ（白/緑）")
logger.warning("警告ログ（黄色）")
logger.error("エラーログ（赤）")
logger.debug("デバッグログ（薄い色。-d オプションで起動時のみ表示）")
```

---

## 12. プラグイン開発チェックリスト

### UIパネルプラグインを作るとき

- [ ] `PLUGIN_ID` を設定した（ユニークな英数字）
- [ ] `PLUGIN_NAME` を設定した（UIに表示される名前）
- [ ] `PLUGIN_TYPE` を `"BACKGROUND"` か `"TOOL"` に設定した
- [ ] `plugins/` フォルダに `.py` ファイルを置いた
- [ ] `start()` でスレッドを立ち上げる場合、`stop()` でフラグを下ろしている
- [ ] `while` ループ内に `time.sleep()` を入れた（忘れるとCPU 100%になる）
- [ ] エラーが起きうる箇所を `try-except` で囲んだ
- [ ] バッジ表示の方針を決めた（`is_connected` 使用 or `enabled` フォールバック）

### CMDコマンドプラグインを作るとき

- [ ] `IDENTIFIER` を設定した（英大文字推奨。`[CMD:XXX]` の XXX の部分）
- [ ] `handle(self, value: str)` を実装した
- [ ] `plugins/` フォルダに `.py` ファイルを置いた
- [ ] 必須設定がある場合は `REQUIRED_CONFIG` にキー名を列挙した

### 共通

- [ ] `from plugin_manager import BasePlugin` でインポートしている
- [ ] `super().__init__()` を `__init__` の先頭で呼んでいる（`PLUGIN_ID` を使う場合は必須）
- [ ] `logger.info` / `logger.warning` でログを出すようにした

---

## 13. よくある失敗と対処法

**UIパネルに表示されない**
→ `PLUGIN_ID = ""` のままになっていないか確認。空だとスキャンされません。

**CMDが反応しない**
→ `IDENTIFIER` が設定されているか確認。また `.py` ファイルが `plugins/` フォルダに置かれているか確認してください。

**配信中にフリーズする**
→ `while self.is_running:` ループ内に `time.sleep(1.0)` がないのが原因。

**スレッドが停止しない**
→ `stop()` でフラグを `False` にしているか確認。スレッドは `daemon=True` にしておくと安全。

**設定が保存されない**
→ `PLUGIN_ID` を設定していないと `save_settings()` が動きません。`super().__init__()` を呼んでいるか確認。

**enabled=True にしてもバッジが光らない**
→ `__init__` で `self.is_connected = False` または `self.is_running = False` を定義していると、`enabled` よりこちらが優先されてバッジがグレーになります。
外部サービス接続不要なプラグインは以下のどちらかにしてください：
- **`is_connected` / `is_running` を定義しない** → `enabled` フォールバックが使われる（シンプルな用途に推奨）
- **`self.is_connected = True` と固定する** → 常にアクティブバッジ（常時アクティブにしたい場合）

`is_connected = False` と初期化して後から `True` に切り替える実装は、外部サービス接続型と同じ挙動（未接続中はバッジがグレー）になります。意図して使う分には正しいパターンです。

**`[CMD:IMG]` の設定が反映されない**
→ `img_plugin.py` が `plugins/` フォルダに置かれているか確認してください。
