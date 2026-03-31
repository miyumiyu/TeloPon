# TeloPon: 플러그인 개발 완전 가이드 🧩

TeloPon 플러그인에는 **2가지 주요 유형**이 있습니다.

| 유형 | 설명 | 등록 방식 |
|------|------|----------|
| **UI 패널 플러그인** | TeloPon의 "확장" 화면에 표시됩니다. AI 데이터 주입 및 설정 패널을 제공합니다. | `plugins/` 폴더에 파일을 배치하면 자동 등록 |
| **CMD 명령 플러그인** | AI의 출력에서 `[CMD:XXX]`라고 말할 때 호출되는 처리를 담당합니다. | `plugins/` 폴더에 파일을 배치하면 자동 등록 |

두 유형 모두 동일한 `BasePlugin` 클래스를 상속하여 만듭니다.
**두 기능을 하나의 클래스로 결합**할 수도 있습니다.

---

## 1. 공통 기반: BasePlugin 클래스

모든 플러그인은 `plugin_manager.py`의 `BasePlugin`을 상속하여 만듭니다.

```python
from plugin_manager import BasePlugin
```

`BasePlugin`에는 UI 패널용과 CMD 명령용 **두 세트의 필드와 메서드**가 있습니다.
사용하려는 것만 설정하고 구현하면 됩니다.

### 클래스 변수

```python
class BasePlugin:
    # ===== UI 패널 플러그인용 =====
    PLUGIN_ID   = ""           # ← 비어 있으면 UI 패널로 등록되지 않음
    PLUGIN_NAME = "Base Plugin"
    PLUGIN_TYPE = "BACKGROUND" # "BACKGROUND" / "TOOL"

    # ===== CMD 명령 플러그인용 =====
    IDENTIFIER      = ""       # ← [CMD:XXX]의 XXX. 비어 있으면 CMD로 등록되지 않음
    REQUIRED_CONFIG: list = [] # setup()에서 미리 확인할 설정 키 이름 목록
```

`PLUGIN_ID` 설정 → UI 패널 플러그인으로 동작
`IDENTIFIER` 설정 → CMD 명령 플러그인으로 동작
**둘 다 설정 → 양쪽 기능을 가진 하이브리드 플러그인이 됨**

---

## 2. 두 가지 PLUGIN_TYPE

UI 패널 플러그인을 만들 때는 `PLUGIN_TYPE`을 다음 중 하나로 설정합니다:

| PLUGIN_TYPE | 설명 |
|------------|------|
| `"BACKGROUND"` | 스트리밍 중에는 설정 화면이 잠깁니다. 백그라운드에서 자동으로 실행되는 시스템용. |
| `"TOOL"` | 스트리밍 중에도 언제든지 제어 패널을 열어 조작할 수 있습니다. 수동 도구용. |

---

## 3. 배지 표시 메커니즘

확장 기능 패널의 각 플러그인에는 **TOOL / ON / OFF** 배지가 표시됩니다.
배지 색상(활성=파란색/녹색 또는 회색)은 다음 우선순위로 결정됩니다.

```
우선순위 1: is_connected 속성이 True  → 활성 (파란색/녹색)
우선순위 2: is_running   속성이 True  → 활성 (파란색/녹색)
폴백: 위 두 속성 모두 없으면 → enabled 설정값 사용
```

### 상태 조합과 결과（전체 패턴）

| `is_connected` | `is_running` | `enabled` | 배지 |
|---|---|---|---|
| True（정의됨） | — | — | 활성（파란색/녹색） |
| False（정의됨） | — | — | 회색 |
| 미정의 | True | — | 활성（파란색/녹색） |
| 미정의 | False | — | 회색 |
| 미정의 | 미정의 | True | 활성（파란색/녹색） |
| 미정의 | 미정의 | False | 회색 |

> **`is_running`은 라이브 세션 이벤트에 연동됩니다.**
> TeloPon이 Gemini Live에 연결·해제될 때 모든 플러그인의 `start()` / `stop()`이 호출됩니다.
> `start()` 안에서 `self.is_running = True`, `stop()` 안에서 `False`로 설정하는 것이 일반적인 사용법입니다.

### 패턴별 동작（용도 빠른 참조）

| 플러그인 구현 | 배지가 활성화되는 조건 | 일반적인 용도 |
|---|---|---|
| `is_connected` 정의됨・True 고정 | 항상 활성 | 외부 접속 불필요하지만 항상 활성 배지를 원하는 TOOL |
| `is_connected` 정의됨・가변 | `is_connected = True` 일 때 | YouTube·Twitch 등 외부 서비스 연결형 |
| `is_running` 정의됨 (`is_connected` 없음) | `is_running = True` 일 때 | 백그라운드 처리형 |
| 둘 다 미정의 | `enabled = True` 일 때 | 단순한 커스텀 플러그인 |

### 중요: `is_connected`를 정의할지 여부

**외부 서비스에 연결하는 플러그인(YouTube·Twitch 등)**은 `is_connected`를 정의하고,
연결 완료 시에만 `True`로 설정합니다. 미연결 상태에서 배지가 켜지는 것은 올바르지 않기 때문입니다.

```python
# 외부 서비스 연결형: is_connected로 제어
class YouTubePlugin(BasePlugin):
    def __init__(self):
        super().__init__()
        self.is_connected = False  # 연결 완료 시 True로 설정
        self.is_running   = False  # 라이브 시작 시 True로 설정
```

**단순한 커스텀 플러그인**은 `is_connected` / `is_running`을 정의하지 않으면
`enabled = True`만으로 배지가 활성화됩니다.

```python
# 단순 커스텀 플러그인: enabled로 제어 (is_connected를 정의하지 않음)
class MySimplePlugin(BasePlugin):
    PLUGIN_ID   = "my_plugin"
    PLUGIN_TYPE = "TOOL"

    def get_default_settings(self):
        return {"enabled": True}  # 이것만으로 라이브 전부터 배지가 켜짐

    # is_connected / is_running은 정의하지 않음 ← 핵심
```

---

## 4. UI 패널 플러그인 만들기

### 파일 배치 위치

```
TeloPon/
 └── plugins/
      └── my_plugin.py   ← 여기에 배치하면 자동으로 UI에 표시됨
```

`PLUGIN_ID`가 설정된 경우, 앱 시작 시 `plugins/` 폴더를 자동으로 스캔하여 등록합니다.

### 최소 예시

```python
import tkinter as tk
from tkinter import ttk
from plugin_manager import BasePlugin
import logger

class MyPlugin(BasePlugin):
    PLUGIN_ID   = "my_plugin"      # ← 고유 ID (파일명과 일치시키는 것이 좋은 관행)
    PLUGIN_NAME = "🔧 샘플 플러그인"
    PLUGIN_TYPE = "BACKGROUND"

    def get_default_settings(self):
        """설정의 기본값을 반환합니다"""
        return {"enabled": False, "message": "안녕하세요"}

    def open_settings_ui(self, parent_window):
        """⚙️ 설정 버튼을 눌렀을 때 열리는 창"""
        panel = tk.Toplevel(parent_window)
        panel.title(self.PLUGIN_NAME)
        panel.geometry("300x150")
        panel.attributes("-topmost", True)

        settings = self.get_settings()

        ttk.Label(panel, text="Message:").pack(pady=5)
        ent = ttk.Entry(panel, width=30)
        ent.insert(0, settings.get("message", ""))
        ent.pack()

        def save():
            s = self.get_settings()
            s["message"] = ent.get()
            self.save_settings(s)
            panel.destroy()

        ttk.Button(panel, text="Save", command=save).pack(pady=10)

    def start(self, prompt_config, plugin_queue):
        """▶️ 라이브 방송 시작 시 호출됨"""
        logger.info(f"[{self.PLUGIN_NAME}] 시작됨")

    def stop(self):
        """⏹️ 라이브 방송 종료 시 호출됨"""
        logger.info(f"[{self.PLUGIN_NAME}] 종료됨")
```

### 설정 파일에 대해

설정은 자동으로 `plugins/my_plugin.json`에 저장됩니다. 파일을 직접 조작할 필요가 없습니다.

```python
# 읽기 (get_default_settings의 값이 기준으로 사용됨)
settings = self.get_settings()
value = settings.get("my_key", "기본값")

# 저장
settings = self.get_settings()
settings["my_key"] = "새로운 값"
self.save_settings(settings)
```

---

## 5. AI에 정보 주입하기 (UI 패널 플러그인의 주요 역할)

### 방법 ① 방송 시작 전에 프롬프트에 추가

```python
def get_prompt_addon(self):
    """방송 시작 직전에 호출됩니다. AI의 시스템 프롬프트에 추가할 문자열을 반환합니다."""
    settings = self.get_settings()
    if not settings.get("enabled"):
        return ""
    return """
    [사전 공지]
    오늘 방송은 시청자 100명 달성 기념입니다.
    AI는 이를 고려하여 처음에 축하 멘트를 포함해야 합니다.
    """
```

### 방법 ② 방송 중 실시간으로 AI에게 텍스트 전송

```python
def start(self, prompt_config, plugin_queue):
    self.plugin_queue = plugin_queue  # 큐를 저장
    # ... 스레드 시작 등

def _some_event_happened(self):
    # AI에게 텍스트 주입 (전용 메서드 사용이 편리함)
    self.send_text(self.plugin_queue, "[알림] 시청자 수가 증가했습니다! 이것을 알려주세요.")
    # 직접 넣어도 결과는 같음
    # self.plugin_queue.put({"type": "text", "content": "..."})
```

### 방법 ③ 방송 중 실시간으로 AI에게 이미지 전송

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

    # 전용 메서드 사용이 편리함
    self.send_image(self.plugin_queue, buf.getvalue())
    # 이미지가 무엇인지에 대한 지시를 보내는 것이 핵심!
    self.send_text(self.plugin_queue, "방금 본 이미지에 대해 코멘트해 주세요.")
```

---

## 6. BACKGROUND 플러그인 전체 템플릿

백그라운드에서 지속적으로 실행되며 주기적으로 AI에게 정보를 전송하는 플러그인의 완전한 템플릿입니다.

```python
import time
import threading
import tkinter as tk
from tkinter import ttk, messagebox
from plugin_manager import BasePlugin
import logger


class AutoTimerPlugin(BasePlugin):
    PLUGIN_ID   = "auto_timer_plugin"
    PLUGIN_NAME = "⌚ 자동 타이머 플러그인"
    PLUGIN_TYPE = "BACKGROUND"

    def __init__(self):
        super().__init__()
        self.plugin_queue = None
        self.is_running   = False
        self.is_connected = False  # UI와 동기화된 연결 상태 플래그 (배지 표시에도 사용됨)
        self.thread       = None

    def get_default_settings(self):
        return {"enabled": False, "interval_sec": 60}

    # ==========================================
    # 설정 UI
    # ==========================================
    def open_settings_ui(self, parent_window):
        # 이미 열려 있으면 앞으로 가져오기
        if hasattr(self, "panel") and self.panel is not None and self.panel.winfo_exists():
            self.panel.lift()
            return

        self.panel = tk.Toplevel(parent_window)
        self.panel.title(f"{self.PLUGIN_NAME} 설정")
        self.panel.geometry("350x220")
        self.panel.attributes("-topmost", True)

        settings = self.get_settings()
        self.is_connected = settings.get("enabled", False)

        main_f = ttk.Frame(self.panel, padding=15)
        main_f.pack(fill=tk.BOTH, expand=True)

        # 상태 레이블
        self.lbl_status = ttk.Label(main_f, text="", font=("", 11, "bold"))
        self.lbl_status.pack(anchor="w", pady=(0, 10))

        # 설정 입력
        row = ttk.Frame(main_f)
        row.pack(fill=tk.X, pady=5)
        ttk.Label(row, text="알림 간격 (초):").pack(side="left")
        self.ent_interval = ttk.Entry(row, width=8)
        self.ent_interval.pack(side="left", padx=5)
        self.ent_interval.insert(0, str(settings.get("interval_sec", 60)))

        # 연결/연결 해제 버튼
        self.btn_connect = tk.Button(
            main_f, font=("", 10, "bold"),
            command=self._toggle_connection
        )
        self.btn_connect.pack(fill=tk.X, pady=8)

        # 저장 후 닫기 버튼
        tk.Button(
            main_f, text="저장 후 닫기",
            bg="#6c757d", fg="white",
            command=self._save_and_close
        ).pack(fill=tk.X)

        self._update_ui()  # 초기 UI 상태 적용

    def _update_ui(self):
        """연결 상태에 따라 UI 모양 전환"""
        if not hasattr(self, "btn_connect") or not self.btn_connect.winfo_exists():
            return
        if self.is_connected:
            self.lbl_status.config(text="⚡ 실행 중", foreground="#28a745")
            self.btn_connect.config(text="정지", bg="#dc3545", fg="white")
            self.ent_interval.config(state="disabled")  # 실행 중에는 잠금
        else:
            self.lbl_status.config(text="🔌 정지됨", foreground="gray")
            self.btn_connect.config(text="시작", bg="#007bff", fg="white")
            self.ent_interval.config(state="normal")

    def _toggle_connection(self):
        if self.is_connected:
            self.is_connected = False
            self._update_ui()
            self._save_from_ui()
            logger.info(f"[{self.PLUGIN_NAME}] 정지됨")
        else:
            try:
                int(self.ent_interval.get())
            except ValueError:
                messagebox.showwarning("오류", "초 값에 숫자를 입력해 주세요")
                return
            self.is_connected = True
            self._update_ui()
            self._save_from_ui()
            logger.info(f"[{self.PLUGIN_NAME}] 활성화됨")

    def _save_from_ui(self):
        s = self.get_settings()          # ← 현재 설정 읽기
        s["enabled"]      = self.is_connected
        s["interval_sec"] = int(self.ent_interval.get())
        self.save_settings(s)

    def _save_and_close(self):
        self._save_from_ui()
        self.panel.destroy()

    # ==========================================
    # 라이프사이클 (라이브 방송 시작/종료)
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
        logger.info(f"[{self.PLUGIN_NAME}] 백그라운드 루프 시작됨")

    def stop(self):
        self.is_running   = False
        self.plugin_queue = None

    # ==========================================
    # 백그라운드 처리 (별도 스레드에서 실행)
    # ==========================================
    def _loop(self, prompt_config):
        settings     = self.get_settings()
        interval     = int(settings.get("interval_sec", 60))
        ai_name      = prompt_config.get("ai_name", "AI")
        elapsed      = 0

        while self.is_running:
            time.sleep(1.0)  # ← 필수! 없으면 CPU 사용률이 100%가 됨
            elapsed += 1

            if elapsed >= interval:
                current_time = time.strftime("%I:%M %p")
                self.send_text(
                    self.plugin_queue,
                    f"[시스템 알림] 현재 시각은 {current_time}입니다. {ai_name}, 시간을 알려주세요."
                )
                logger.info(f"[{self.PLUGIN_NAME}] 시간을 AI에게 전송: {current_time}")
                elapsed = 0
```

---

## 7. TOOL 플러그인 템플릿

스트리밍 중에도 조작할 수 있는 수동 도구입니다. `PLUGIN_TYPE = "TOOL"`로 설정하기만 하면 스트리밍 중에 패널을 조작할 수 있습니다.

### 단순한 TOOL형 (enabled만으로 배지 제어)

`is_connected` / `is_running`을 정의하지 않는 단순한 구성입니다.
`enabled = True` 상태로 배지가 활성화되므로, 라이브 전부터 파란색으로 켜집니다.

```python
import tkinter as tk
from tkinter import ttk, messagebox
from plugin_manager import BasePlugin
import logger


class ManualCuePlugin(BasePlugin):
    PLUGIN_ID   = "manual_cue_plugin"
    PLUGIN_NAME = "🎬 수동 큐 발송기"
    PLUGIN_TYPE = "TOOL"  # ← 이것이 핵심

    def __init__(self):
        super().__init__()
        self.plugin_queue = None
        # ★ is_connected / is_running을 정의하지 않음
        #   → get_default_settings의 enabled 값으로 배지 제어됨

    def get_default_settings(self):
        return {"enabled": True}  # True이면 라이브 전부터 배지가 파란색으로 켜짐

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

        ttk.Label(self.panel, text="AI에게 보낼 메시지", font=("", 11)).pack(pady=10)

        ent = ttk.Entry(self.panel, width=35)
        ent.pack(pady=5)
        ent.insert(0, "지금 당장 웃음을 가져오세요!")

        lbl = ttk.Label(self.panel, text="")
        lbl.pack(pady=5)

        def send():
            if not self.plugin_queue:
                messagebox.showwarning("경고", "이것을 누르기 전에 먼저 방송을 시작해 주세요")
                return
            msg = ent.get().strip()
            if not msg:
                return
            self.send_text(self.plugin_queue, f"[디렉터 지시] {msg}")
            lbl.config(text="✅ 전송 완료!", foreground="green")
            logger.info(f"[{self.PLUGIN_NAME}] 수동 큐 전송: {msg}")

        ttk.Button(self.panel, text="📨 AI에게 전송", command=send).pack(pady=10)
```

### is_connected=True 고정 패턴（상시 활성형）

외부 서비스 연결이 불필요하지만 `enabled` 설정에 관계없이 배지를 항상 활성화하려면
`is_connected = True`를 고정으로 설정합니다.

```python
class AlwaysActiveToolPlugin(BasePlugin):
    PLUGIN_ID   = "always_active_plugin"
    PLUGIN_NAME = "🔧 상시 활성 플러그인"
    PLUGIN_TYPE = "TOOL"

    def __init__(self):
        super().__init__()
        self.is_connected = True  # 고정 True → 배지가 항상 활성
        self.plugin_queue = None

    def get_default_settings(self):
        return {"enabled": True}

    def start(self, prompt_config, plugin_queue):
        self.plugin_queue = plugin_queue

    def stop(self):
        self.plugin_queue = None
```

---

## 8. CMD 명령 플러그인 만들기

AI가 출력에서 `[CMD:XXX] 인수`라고 쓸 때 처리를 실행하는 플러그인입니다.

### 동작 방식

```
AI 출력:  "이제 이미지를 표시하겠습니다 [CMD:IMG] 석양 풍경 생성"
              ↓
TeloPon이 [CMD:IMG]를 감지
              ↓
ImgPlugin의 handle("석양 풍경 생성")이 호출됨
```

### 최소 예시

```python
from plugin_manager import BasePlugin
import logger


class MyCommandPlugin(BasePlugin):
    IDENTIFIER = "MYCMD"   # ← [CMD:MYCMD]에 반응. PLUGIN_ID는 설정하지 않음
    REQUIRED_CONFIG = []

    def handle(self, value: str):
        """[CMD:MYCMD] 이후의 부분이 value로 전달됨"""
        logger.info(f"[CMD:MYCMD] 수신: {value}")
        # 처리 내용을 여기에 작성


plugin = MyCommandPlugin()
```

### 자동 등록 메커니즘

CMD 플러그인은 **`plugins/` 폴더에 `.py` 파일을 배치하기만 하면 자동 등록됩니다**.
`IDENTIFIER`가 설정되어 있으면 앱 시작 시 자동으로 스캔됩니다.

```
TeloPon/
 └── plugins/
      └── my_command_plugin.py   ← 여기에 배치하면 [CMD:MYCMD]가 활성화됨
```

### REQUIRED_CONFIG 사용 방법

플러그인이 특정 설정(API 키 등) 없이는 동작할 수 없는 경우, `REQUIRED_CONFIG`에 설정 키를 나열합니다.
설정되지 않은 경우 플러그인이 자동으로 건너뛰어져 오류를 방지합니다.

```python
class WeatherPlugin(BasePlugin):
    IDENTIFIER      = "WEATHER"
    REQUIRED_CONFIG = ["weather_api_key"]  # config.weather_api_key가 필요

    def handle(self, value: str):
        import config
        api_key = config.app_config["weather_api_key"]
        # ...
```

---

## 9. 하이브리드 플러그인: UI 패널과 CMD 결합

`PLUGIN_ID`와 `IDENTIFIER`를 모두 설정하면 UI 패널 기능과 CMD 명령 기능을 모두 가진 플러그인이 됩니다.

```python
from plugin_manager import BasePlugin
import logger


class HybridPlugin(BasePlugin):
    # UI 패널로 등록
    PLUGIN_ID   = "hybrid_plugin"
    PLUGIN_NAME = "🔀 하이브리드 플러그인"
    PLUGIN_TYPE = "TOOL"

    # CMD 명령으로도 등록
    IDENTIFIER = "HYBRID"

    def __init__(self):
        super().__init__()
        self.plugin_queue = None

    # ----- UI 패널 측 -----
    def start(self, prompt_config, plugin_queue):
        self.plugin_queue = plugin_queue

    def stop(self):
        self.plugin_queue = None

    def open_settings_ui(self, parent_window):
        # ... 설정 패널 UI

    # ----- CMD 명령 측 -----
    def handle(self, value: str):
        """[CMD:HYBRID] 수신 시 호출되는 처리"""
        logger.info(f"[CMD:HYBRID] 수신: {value}")
        if self.plugin_queue:
            self.send_text(self.plugin_queue, f"하이브리드 처리: {value}")
```

하이브리드 플러그인도 `plugins/` 폴더에 배치하기만 하면 자동 등록됩니다.

---

## 10. 기존 플러그인 예시: img_plugin.py

실제로 사용 중인 CMD 플러그인 예시입니다 (`plugins/img_plugin.py`).

```
[CMD:IMG] 석양 고양이 생성   → Gemini AI를 사용하여 이미지 생성 및 표시
[CMD:IMG] title               → TITLE.png를 대형 형식으로 표시
[CMD:IMG] last                → 이전에 생성된 이미지를 다시 표시
[CMD:IMG] hide                → 이미지 숨기기
[CMD:IMG] big                 → 이미지를 대형 형식으로 표시
[CMD:IMG] small               → 이미지를 축소 크기로 표시
```

`ImgPlugin`은 CMD 전용이므로 `PLUGIN_ID`가 없어 (UI 패널에 표시되지 않음),
`IDENTIFIER = "IMG"`만 설정되어 있습니다.

---

## 11. 자주 사용되는 메서드 및 유틸리티 요약

### send_text / send_image (BasePlugin 내장)

```python
# AI에게 텍스트 전송
self.send_text(self.plugin_queue, "AI에게 보낼 메시지")

# AI에게 이미지 전송 (바이트로 전달)
self.send_image(self.plugin_queue, img_bytes, mime_type="image/jpeg")
```

### logger (TeloPon 표준 로그 출력)

```python
import logger

logger.info("일반 로그 (흰색/녹색)")
logger.warning("경고 로그 (노란색)")
logger.error("오류 로그 (빨간색)")
logger.debug("디버그 로그 (흐린 색상. -d 옵션으로 실행할 때만 표시됨)")
```

---

## 12. 플러그인 개발 체크리스트

### UI 패널 플러그인 만들 때

- [ ] `PLUGIN_ID` 설정 (고유한 영숫자 문자열)
- [ ] `PLUGIN_NAME` 설정 (UI에 표시되는 이름)
- [ ] `PLUGIN_TYPE`을 `"BACKGROUND"` 또는 `"TOOL"`로 설정
- [ ] `plugins/` 폴더에 `.py` 파일 배치
- [ ] `start()`에서 스레드를 시작한 경우, `stop()`에서 플래그를 False로 설정
- [ ] `while` 루프 내부에 `time.sleep()` 추가 (잊으면 CPU 사용률 100%)
- [ ] 오류가 발생할 수 있는 코드를 `try-except`로 감쌈
- [ ] 배지 표시 방침 결정 (`is_connected` 사용 또는 `enabled` 폴백)

### CMD 명령 플러그인 만들 때

- [ ] `IDENTIFIER` 설정 (대문자 권장; `[CMD:XXX]`의 XXX)
- [ ] `handle(self, value: str)` 구현
- [ ] `plugins/` 폴더에 `.py` 파일 배치
- [ ] 필수 설정이 있으면 `REQUIRED_CONFIG`에 키 이름 나열

### 공통

- [ ] `from plugin_manager import BasePlugin`으로 임포트
- [ ] `__init__` 시작 시 `super().__init__()` 호출 (`PLUGIN_ID` 사용 시 필수)
- [ ] 로그 출력에 `logger.info` / `logger.warning` 사용

---

## 13. 흔한 실수와 해결 방법

**플러그인이 UI 패널에 표시되지 않음**
→ `PLUGIN_ID`가 아직 `""`로 설정되어 있지 않은지 확인하세요. 비어 있으면 스캔되지 않습니다.

**CMD가 반응하지 않음**
→ `IDENTIFIER`가 설정되어 있는지 확인하세요. 또한 `.py` 파일이 `plugins/` 폴더에 배치되어 있는지 확인하세요.

**스트리밍 중 시스템이 멈춤**
→ `while self.is_running:` 루프 내부에 `time.sleep(1.0)`이 빠진 것이 원인입니다.

**스레드가 멈추지 않음**
→ `stop()`에서 플래그를 `False`로 설정하는지 확인하세요. 스레드를 `daemon=True`로 설정하는 것이 안전한 관행입니다.

**설정이 저장되지 않음**
→ `PLUGIN_ID`가 설정되지 않으면 `save_settings()`가 작동하지 않습니다. `super().__init__()`이 호출되고 있는지 확인하세요.

**enabled=True로 해도 배지가 켜지지 않음**
→ `__init__`에서 `self.is_connected = False` 또는 `self.is_running = False`를 정의하면 `enabled`보다 이것이 우선되어 배지가 회색이 됩니다.
외부 서비스 연결이 필요 없는 플러그인은 다음 중 하나를 선택하세요：
- **`is_connected` / `is_running`을 정의하지 않음** → `enabled` 폴백이 사용됨（단순한 용도에 권장）
- **`self.is_connected = True`로 고정** → 항상 활성 배지（항상 켜두고 싶은 경우）

`is_connected = False`로 초기화하고 나중에 `True`로 전환하는 구현은 외부 서비스 연결형과 같은 동작（미연결 중에는 배지가 회색）이 됩니다. 의도적으로 사용하는 경우는 올바른 패턴입니다.

**`[CMD:IMG]` 설정이 적용되지 않음**
→ `img_plugin.py`가 `plugins/` 폴더에 배치되어 있는지 확인하세요.
