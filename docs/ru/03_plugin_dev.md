# TeloPon: Полное руководство по разработке плагинов 🧩

Плагины TeloPon бывают **2 основных типов**.

| Тип | Описание | Регистрация |
|------|------|----------|
| **Плагин панели интерфейса** | Отображается на экране «Расширения» TeloPon. Предоставляет панели для вставки данных в ИИ и настройки. | Автоматически регистрируется при размещении файла в папке `plugins/` |
| **Плагин команд CMD** | Обрабатывает процессы, вызываемые когда ИИ пишет `[CMD:XXX]` в своём выводе. | Автоматически регистрируется при размещении файла в папке `plugins/` |

Оба типа создаются путём наследования от одного базового класса `BasePlugin`.
Вы также можете **объединить обе функции в одном классе**.

---

## 1. Общая основа: класс BasePlugin

Все плагины создаются путём наследования от `BasePlugin` в `plugin_manager.py`.

```python
from plugin_manager import BasePlugin
```

`BasePlugin` имеет **два набора полей и методов** — один для панелей интерфейса, один для команд CMD.
Вам нужно только настроить и реализовать те, которые хотите использовать.

### Переменные класса

```python
class BasePlugin:
    # ===== Для плагинов панели интерфейса =====
    PLUGIN_ID   = ""           # ← Если оставить пустым, не будет зарегистрирован как панель интерфейса
    PLUGIN_NAME = "Base Plugin"
    PLUGIN_TYPE = "BACKGROUND" # "BACKGROUND" / "TOOL"

    # ===== Для плагинов команд CMD =====
    IDENTIFIER      = ""       # ← XXX в [CMD:XXX]. Если пусто, не регистрируется как CMD
    REQUIRED_CONFIG: list = [] # Список имён ключей конфигурации для предварительной проверки в setup()
```

Установка `PLUGIN_ID` → работает как плагин панели интерфейса
Установка `IDENTIFIER` → работает как плагин команд CMD
**Установка обоих → становится гибридным плагином с обеими функциями**

---

## 2. Два типа PLUGIN_TYPE

При создании плагина панели интерфейса установите `PLUGIN_TYPE` в одно из следующих значений:

| PLUGIN_TYPE | Описание |
|------------|------|
| `"BACKGROUND"` | Экран настроек заблокирован во время стриминга. Для систем, работающих автоматически в фоне. |
| `"TOOL"` | Панель управления можно открывать и использовать в любое время, даже во время стриминга. Для ручных инструментов. |

---

## 3. Механизм отображения значков

На каждом плагине в панели расширений отображается значок **TOOL / ON / OFF**.
Цвет значка (активный=синий/зелёный или серый) определяется следующим приоритетом:

```
Приоритет 1: атрибут is_connected равен True  → Активен (синий/зелёный)
Приоритет 2: атрибут is_running   равен True  → Активен (синий/зелёный)
Запасной вариант: ни один из атрибутов не определён → используется значение enabled
```

### Все комбинации состояний

| `is_connected` | `is_running` | `enabled` | Значок |
|---|---|---|---|
| True (определён) | — | — | Активен (синий/зелёный) |
| False (определён) | — | — | Серый |
| Не определён | True | — | Активен (синий/зелёный) |
| Не определён | False | — | Серый |
| Не определён | Не определён | True | Активен (синий/зелёный) |
| Не определён | Не определён | False | Серый |

> **`is_running` реагирует на события живой сессии.**
> Когда TeloPon подключается или отключается от Gemini Live, у всех плагинов вызываются `start()` / `stop()`.
> Установка `self.is_running = True` внутри `start()` и `False` внутри `stop()` — типичный паттерн использования.

### Поведение по паттернам (краткий справочник)

| Реализация плагина | Условие активации значка | Типичное использование |
|---|---|---|
| `is_connected` определён, зафиксирован True | Всегда активен | TOOL плагины без внешнего подключения, но с постоянно активным значком |
| `is_connected` определён, переменный | Когда `is_connected = True` | YouTube·Twitch и другие подключения к внешним сервисам |
| `is_running` определён (без `is_connected`) | Когда `is_running = True` | Фоновая обработка |
| Ни один атрибут не определён | Когда `enabled = True` | Простые пользовательские плагины |

### Важно: нужно ли определять `is_connected`

**Плагины, подключающиеся к внешним сервисам (YouTube·Twitch и т.д.)**, должны определять `is_connected`
и устанавливать его в `True` только при успешном подключении. Светящийся значок без подключения неправильен.

```python
# Тип с подключением к внешнему сервису: управление через is_connected
class YouTubePlugin(BasePlugin):
    def __init__(self):
        super().__init__()
        self.is_connected = False  # Установить True при успешном подключении
        self.is_running   = False  # Установить True при начале прямого эфира
```

**Простые пользовательские плагины** могут не определять `is_connected` / `is_running` —
тогда значок активируется просто при `enabled = True`.

```python
# Простой пользовательский плагин: управление через enabled (is_connected не определяется)
class MySimplePlugin(BasePlugin):
    PLUGIN_ID   = "my_plugin"
    PLUGIN_TYPE = "TOOL"

    def get_default_settings(self):
        return {"enabled": True}  # Этого достаточно для активации значка ещё до начала прямого эфира

    # is_connected / is_running не определяются ← Ключевой момент
```

---

## 4. Как создать плагин панели интерфейса

### Где разместить файл

```
TeloPon/
 └── plugins/
      └── my_plugin.py   ← Простое размещение здесь автоматически отображает его в интерфейсе
```

Если задан `PLUGIN_ID`, папка `plugins/` автоматически сканируется и регистрируется при запуске приложения.

### Минимальный пример

```python
import tkinter as tk
from tkinter import ttk
from plugin_manager import BasePlugin
import logger

class MyPlugin(BasePlugin):
    PLUGIN_ID   = "my_plugin"      # ← Уникальный ID (совпадение с именем файла — хорошая практика)
    PLUGIN_NAME = "🔧 Образцовый плагин"
    PLUGIN_TYPE = "BACKGROUND"

    def get_default_settings(self):
        """Возвращает значения по умолчанию для настроек"""
        return {"enabled": False, "message": "Привет"}

    def open_settings_ui(self, parent_window):
        """Окно, открывающееся при нажатии кнопки ⚙️ Настройки"""
        panel = tk.Toplevel(parent_window)
        panel.title(self.PLUGIN_NAME)
        panel.geometry("300x150")
        panel.attributes("-topmost", True)

        settings = self.get_settings()

        ttk.Label(panel, text="Сообщение:").pack(pady=5)
        ent = ttk.Entry(panel, width=30)
        ent.insert(0, settings.get("message", ""))
        ent.pack()

        def save():
            s = self.get_settings()
            s["message"] = ent.get()
            self.save_settings(s)
            panel.destroy()

        ttk.Button(panel, text="Сохранить", command=save).pack(pady=10)

    def start(self, prompt_config, plugin_queue):
        """▶️ Вызывается при начале прямого эфира"""
        logger.info(f"[{self.PLUGIN_NAME}] Запущен")

    def stop(self):
        """⏹️ Вызывается при остановке прямого эфира"""
        logger.info(f"[{self.PLUGIN_NAME}] Остановлен")
```

### О файлах настроек

Настройки автоматически сохраняются в `plugins/my_plugin.json`. Вам не нужно вручную управлять файлами.

```python
# Чтение (значения из get_default_settings используются как база)
settings = self.get_settings()
value = settings.get("my_key", "значение по умолчанию")

# Сохранение
settings = self.get_settings()
settings["my_key"] = "новое значение"
self.save_settings(settings)
```

---

## 5. Вставка информации в ИИ (основная роль плагинов панели интерфейса)

### Метод ① Добавление к промпту перед началом стрима

```python
def get_prompt_addon(self):
    """Вызывается непосредственно перед началом стрима. Возвращает строку для добавления к системному промпту ИИ."""
    settings = self.get_settings()
    if not settings.get("enabled"):
        return ""
    return """
    [Предварительное уведомление]
    Сегодняшний стрим — это празднование достижения 100 зрителей.
    ИИ должен включать поздравительные комментарии в начале, принимая это во внимание.
    """
```

### Метод ② Отправка текста ИИ в реальном времени во время стрима

```python
def start(self, prompt_config, plugin_queue):
    self.plugin_queue = plugin_queue  # Сохранить очередь
    # ... запустить потоки и т.д.

def _some_event_happened(self):
    # Вставить текст ИИ (удобно использовать специальный метод)
    self.send_text(self.plugin_queue, "[Уведомление] Количество зрителей увеличилось! Пожалуйста, подтвердите это.")
    # Или положить напрямую — тот же результат
    # self.plugin_queue.put({"type": "text", "content": "..."})
```

### Метод ③ Отправка изображений ИИ в реальном времени во время стрима

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

    # Удобно использовать специальный метод (mime_type по умолчанию "image/jpeg", если не указан)
    self.send_image(self.plugin_queue, buf.getvalue())
    # Затем отправить инструкции о том, что это за изображение — это ключевой момент!
    self.send_text(self.plugin_queue, "Пожалуйста, прокомментируй изображение, которое ты только что увидел.")
```

---

## 6. Хуки плагина (вывод телопа / получение комментария)

Предоставляются два **хука** для слабосвязанного взаимодействия плагинов.

| Хук | Когда вызывается | Применение |
|---|---|---|
| `on_telop_output(topic, main, window, layout, badge)` | Когда ИИ выводит телоп | Логирование/запись/задержка показа телопа |
| `on_comment_received(text, author, source)` | Когда стриминг-платформа получает комментарий | Счётчик новичков, обнаружение запрещённых слов, аналитика комментариев |

Оба рассылаются **fire-and-forget** (асинхронно, без ожидания) всем плагинам. Ядро не ждёт завершения обработки в плагине.

---

### 6-A. Хук вывода телопов (on_telop_output)

Переопределите метод `on_telop_output`, чтобы получать данные каждый раз, когда ИИ выводит телоп.
Вывод телопов доставляется по принципу **fire-and-forget (отправил и забыл)** -- Core не ждёт завершения обработки в плагине.

### Архитектура (слабосвязанный дизайн)

```
Core (при выводе телопа):
  ① on_telop_output() асинхронно отправляется всем плагинам (fire-and-forget)
  ② Читает значение задержки, заранее установленное через set_telop_delay()
  ③ Управляет отображением телопа в OBS (мгновенно / с задержкой / подавление)
```

**Важно**: Core никогда не ждёт завершения `on_telop_output()`. Что бы ни происходило внутри плагина (ошибки, задержки, зависания и т.д.), это не влияет на Core.

### Две функции, доступные плагинам

| Метод | Назначение | Когда вызывать |
|---|---|---|
| `on_telop_output(topic, main, window, layout, badge)` | Получение данных телопа | Вызывается автоматически Core (возвращаемое значение не требуется) |
| `set_telop_delay(n)` | Установка задержки отображения в OBS | Вызывается плагином в любой момент |

### Управление задержкой (set_telop_delay)

```python
self.set_telop_delay(0)   # Мгновенное отображение (по умолчанию)
self.set_telop_delay(5)   # Отображение через 5 секунд
self.set_telop_delay(-1)  # Подавление отображения в OBS
```

Core считывает `get_telop_delay()` у всех плагинов перед постановкой в очередь и агрегирует значения.
**Правило конфликта нескольких плагинов**: `-1` (подавление) имеет наивысший приоритет. В остальных случаях используется `max(delay)`.

### Получаемые данные

```python
def on_telop_output(self, topic, main, window, layout, badge):
    # Обработка данных (возвращаемое значение не требуется)
    pass
```

| Аргумент | Содержание | Пример |
|---|---|---|
| `topic` | Текст верхней строки | `Для Овечки` |
| `main` | Основной текст | `Какой <b1>наблюдательный</b1> глаз!` |
| `window` | Тип окна | `window-reply`, `window-simple`, `window-explain` |
| `layout` | Макет | `layout-flat`, `layout-big-top` |
| `badge` | Значок (имя автора и т.д.) | `@sheep-x9y`, `NONE` |

**Теги в `main` / `topic`** (необработанные данные до `drawing.apply_semantic_classes`):

| Тег | Значение | Пример |
|---|---|---|
| `<b1>...</b1>` | Цвет выделения 1 (цвет h1 темы) | `<b1>важно</b1>` |
| `<b2>...</b2>` | Цвет выделения 2 (цвет h2 темы) | `<b2>внимание</b2>` |
| `<P1>` `<M5>` и т.д. | Плитки маджонга | `<P1><P1><P1>` |

### Пример: Плагин-монитор (без управления отображением)

```python
def on_telop_output(self, topic, main, window, layout, badge):
    # Просто записать содержимое телопа в лог
    logger.info(f"[Monitor] {window} | {topic} | {main}")
```

### Пример: Настройка задержки

```python
def __init__(self):
    super().__init__()
    self.set_telop_delay(5)  # Начальное значение: задержка 5 секунд

def on_telop_output(self, topic, main, window, layout, badge):
    # Получение данных (задержка уже настроена через set_telop_delay)
    self._log_telop(topic, main)

def _on_delay_changed(self, new_value):
    # Вызывается при изменении значения задержки из UI
    self.set_telop_delay(new_value)
```

### Пример: Условное подавление

```python
def __init__(self):
    super().__init__()
    self._suppress_reply = False

def on_telop_output(self, topic, main, window, layout, badge):
    # Данные всегда принимаются
    self._process(topic, main)

def toggle_suppress(self):
    # Вызывается из чекбокса в UI
    self._suppress_reply = not self._suppress_reply
    self.set_telop_delay(-1 if self._suppress_reply else 0)
```

### ⚠️ Важно: Потокобезопасность

`on_telop_output` вызывается из **фонового потока** (асинхронное выполнение в daemon-потоке).
Действуют следующие ограничения.

**🚫 Запрещено: прямые операции с Tkinter**

Вызов методов Tkinter напрямую внутри `on_telop_output` вызывает **взаимную блокировку (deadlock)**:

```python
# ❌ НЕ вызывайте это внутри on_telop_output
self._panel.after(0, ...)        # Deadlock
self._panel.winfo_exists()       # Deadlock
self._text_widget.insert(...)    # Deadlock
```

**✅ Правильный подход: буфер deque + опрос**

Добавляйте данные в потокобезопасный `deque`, затем обрабатывайте в таймере главного потока.

```python
from collections import deque

class MyPlugin(BasePlugin):
    def __init__(self):
        super().__init__()
        self._log_buffer = deque(maxlen=500)  # Потокобезопасный

    def on_telop_output(self, topic, main, window, layout, badge):
        # ✅ Только операции с deque (без Tkinter)
        self._log_buffer.append((topic, main, window))

    def open_settings_ui(self, parent_window):
        self._panel = tk.Toplevel(parent_window)
        # ... построение UI ...
        self._poll()  # Запуск опроса

    def _poll(self):
        # ✅ Потребление deque и обновление UI в главном потоке
        while self._log_buffer:
            topic, main, window = self._log_buffer.popleft()
            self._text_widget.insert("end", f"{topic} | {main}\n")
        if self._panel and self._panel.winfo_exists():
            self._panel.after(100, self._poll)
```

**Безопасные операции внутри `on_telop_output`:**
- Чтение/запись переменных (`self._delay_value` и т.д.)
- `deque.append()` / `list.append()` / `queue.put()`
- Файловый ввод/вывод
- Сетевые вызовы
- Тяжёлые/долгие операции (не влияют на Core)

### Атрибут ALWAYS_ACTIVE

Установите `ALWAYS_ACTIVE = True`, чтобы плагин всегда отображался активным цветом (чёрный текст, жирный шрифт) в списке плагинов.
Хук `on_telop_output` вызывается для всех плагинов вне зависимости от состояния `enabled`, что делает его идеальным для плагинов-мониторов, которые работают просто при открытии панели.

```python
class MyViewerPlugin(BasePlugin):
    PLUGIN_ID   = "my_viewer"
    PLUGIN_TYPE = "TOOL"
    ALWAYS_ACTIVE = True  # Всегда отображается как активный
```

---

### 6-B. Хук получения комментария (on_comment_received)

Когда плагин стриминг-платформы (YouTube, Twitch, Niconico и т.п.) получает комментарий зрителя, уведомление приходит всем активным плагинам.

#### Архитектура

```
Плагин стриминг-платформы (например, YouTube): получен комментарий
       ↓
 self.broadcast_comment(text, author, source)
       ↓
PluginManager: асинхронно рассылает всем активным плагинам
       ↓
on_comment_received() других плагинов вызывается
```

#### Сторона отправителя (плагин, который рассылает комментарии)

Плагин стриминг-платформы, получивший комментарий, вызывает `BasePlugin.broadcast_comment()` для уведомления других плагинов.

```python
def _on_comment(self, text, author):
    # Существующая обработка: отправка ИИ и т.п.
    self.send_text(self.plugin_queue, f"[COMMENT]{author}: {text}")
    # Уведомить другие плагины (fire-and-forget)
    self.broadcast_comment(text, author, source="youtube")
```

| Аргумент | Содержание | Пример |
|---|---|---|
| `text` | Текст комментария | `Я впервые здесь!` |
| `author` | Имя автора | `ЗрительA` |
| `source` | Идентификатор платформы | `"youtube"` / `"twitch"` / `"niconico"` |

#### Сторона получателя (плагин, который потребляет комментарии)

```python
class FirstTimerPlugin(BasePlugin):
    PLUGIN_ID   = "first_timer"
    PLUGIN_TYPE = "TOOL"

    def __init__(self):
        super().__init__()
        self._known_users = set()
        self._count = 0

    def on_comment_received(self, text, author, source):
        """Получает комментарии со всех платформ (без возврата)"""
        if author in self._known_users:
            return
        self._known_users.add(author)
        if any(kw in text.lower() for kw in ["впервые", "привет", "здравствуйте"]):
            self._count += 1
            logger.info(f"[FirstTimer] +1 → {self._count} новичков")
```

#### ⚠️ Потокобезопасность

`on_comment_received` также вызывается из **фонового потока**. Прямые операции Tkinter запрещены (то же ограничение, что и для `on_telop_output`).
Для обновления UI используйте паттерн **deque-буфер + опрос** (см. раздел "⚠️ Важно: потокобезопасность" выше).

#### Существующие плагины-отправители

Плагины стриминг-платформ, которые уже вызывают `broadcast_comment()`:

| Плагин | Значение source |
|---|---|
| YoutubeLivePlugin (встроенный) | `"youtube"` |
| YoutubeLiveOAuth (расширение) | `"youtube"` |
| TwitchPlugin (встроенный) | `"twitch"` |
| NiconicoLivePlugin (расширение) | `"niconico"` |

При создании нового плагина стриминг-платформы вызов `broadcast_comment()` сделает его доступным для других плагинов.

#### Примеры применения

- **Счётчик новичков**: обнаружение фраз «впервые», «привет» и подсчёт уникальных зрителей (показ постоянно через `window-status`)
- **Обнаружение запрещённых слов**: логирование неуместных сообщений
- **Аналитика комментариев**: подсчёт частоты ключевых слов
- **Сбор комментариев**: агрегирование комментариев с нескольких платформ в одном месте

---

## 7. Полный шаблон плагина BACKGROUND

Полный шаблон плагина, который непрерывно работает в фоне и периодически отправляет информацию ИИ.

```python
import time
import threading
import tkinter as tk
from tkinter import ttk, messagebox
from plugin_manager import BasePlugin
import logger


class AutoTimerPlugin(BasePlugin):
    PLUGIN_ID   = "auto_timer_plugin"
    PLUGIN_NAME = "⌚ Плагин автотаймера"
    PLUGIN_TYPE = "BACKGROUND"

    def __init__(self):
        super().__init__()
        self.plugin_queue = None
        self.is_running   = False
        self.is_connected = False  # Флаг состояния соединения, синхронизированный с интерфейсом (используется для значка)
        self.thread       = None

    def get_default_settings(self):
        return {"enabled": False, "interval_sec": 60}

    # ==========================================
    # Интерфейс настроек
    # ==========================================
    def open_settings_ui(self, parent_window):
        # Если уже открыто, переместить на передний план
        if hasattr(self, "panel") and self.panel is not None and self.panel.winfo_exists():
            self.panel.lift()
            return

        self.panel = tk.Toplevel(parent_window)
        self.panel.title(f"Настройки {self.PLUGIN_NAME}")
        self.panel.geometry("350x220")
        self.panel.attributes("-topmost", True)

        settings = self.get_settings()
        self.is_connected = settings.get("enabled", False)

        main_f = ttk.Frame(self.panel, padding=15)
        main_f.pack(fill=tk.BOTH, expand=True)

        # Метка статуса
        self.lbl_status = ttk.Label(main_f, text="", font=("", 11, "bold"))
        self.lbl_status.pack(anchor="w", pady=(0, 10))

        # Ввод настроек
        row = ttk.Frame(main_f)
        row.pack(fill=tk.X, pady=5)
        ttk.Label(row, text="Интервал уведомлений (секунды):").pack(side="left")
        self.ent_interval = ttk.Entry(row, width=8)
        self.ent_interval.pack(side="left", padx=5)
        self.ent_interval.insert(0, str(settings.get("interval_sec", 60)))

        # Кнопка подключения/отключения
        self.btn_connect = tk.Button(
            main_f, font=("", 10, "bold"),
            command=self._toggle_connection
        )
        self.btn_connect.pack(fill=tk.X, pady=8)

        # Кнопка сохранения и закрытия
        tk.Button(
            main_f, text="Сохранить и закрыть",
            bg="#6c757d", fg="white",
            command=self._save_and_close
        ).pack(fill=tk.X)

        self._update_ui()  # Применить начальное состояние интерфейса

    def _update_ui(self):
        """Переключить внешний вид интерфейса в зависимости от состояния соединения"""
        if not hasattr(self, "btn_connect") or not self.btn_connect.winfo_exists():
            return
        if self.is_connected:
            self.lbl_status.config(text="⚡ Работает", foreground="#28a745")
            self.btn_connect.config(text="Остановить", bg="#dc3545", fg="white")
            self.ent_interval.config(state="disabled")  # Заблокировать во время работы
        else:
            self.lbl_status.config(text="🔌 Остановлен", foreground="gray")
            self.btn_connect.config(text="Запустить", bg="#007bff", fg="white")
            self.ent_interval.config(state="normal")

    def _toggle_connection(self):
        if self.is_connected:
            self.is_connected = False
            self._update_ui()
            self._save_from_ui()
            logger.info(f"[{self.PLUGIN_NAME}] Остановлен")
        else:
            try:
                int(self.ent_interval.get())
            except ValueError:
                messagebox.showwarning("Ошибка", "Пожалуйста, введите число для значения секунд")
                return
            self.is_connected = True
            self._update_ui()
            self._save_from_ui()
            logger.info(f"[{self.PLUGIN_NAME}] Включён")

    def _save_from_ui(self):
        s = self.get_settings()          # ← Считать текущие настройки
        s["enabled"]      = self.is_connected
        s["interval_sec"] = int(self.ent_interval.get())
        self.save_settings(s)

    def _save_and_close(self):
        self._save_from_ui()
        self.panel.destroy()

    # ==========================================
    # Жизненный цикл (запуск/остановка прямого эфира)
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
        logger.info(f"[{self.PLUGIN_NAME}] Фоновый цикл запущен")

    def stop(self):
        self.is_running   = False
        self.plugin_queue = None

    # ==========================================
    # Фоновая обработка (выполняется в отдельном потоке)
    # ==========================================
    def _loop(self, prompt_config):
        settings     = self.get_settings()
        interval     = int(settings.get("interval_sec", 60))
        ai_name      = prompt_config.get("ai_name", "AI")
        elapsed      = 0

        while self.is_running:
            time.sleep(1.0)  # ← Обязательно! Без этого загрузка CPU достигнет 100%
            elapsed += 1

            if elapsed >= interval:
                current_time = time.strftime("%H:%M")
                self.send_text(
                    self.plugin_queue,
                    f"[Системное уведомление] Текущее время {current_time}. {ai_name}, пожалуйста, объяви время."
                )
                logger.info(f"[{self.PLUGIN_NAME}] Время отправлено ИИ: {current_time}")
                elapsed = 0
```

---

## 8. Шаблон плагина TOOL

Ручной инструмент, которым можно пользоваться даже во время стриминга. Простая установка `PLUGIN_TYPE = "TOOL"` делает панель работоспособной во время стримов.

### Простой тип TOOL (управление значком только через enabled)

Простая конфигурация без определения `is_connected` / `is_running`.
Значок активируется при `enabled = True`, то есть светится синим ещё до начала прямого эфира.

```python
import tkinter as tk
from tkinter import ttk, messagebox
from plugin_manager import BasePlugin
import logger


class ManualCuePlugin(BasePlugin):
    PLUGIN_ID   = "manual_cue_plugin"
    PLUGIN_NAME = "🎬 Ручная отправка реплики"
    PLUGIN_TYPE = "TOOL"  # ← Это ключевой момент

    def __init__(self):
        super().__init__()
        self.plugin_queue = None
        # ★ is_connected / is_running не определяются
        #   → значок управляется значением enabled из get_default_settings

    def get_default_settings(self):
        return {"enabled": True}  # True — значок светится синим ещё до прямого эфира

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

        ttk.Label(self.panel, text="Сообщение для отправки ИИ", font=("", 11)).pack(pady=10)

        ent = ttk.Entry(self.panel, width=35)
        ent.pack(pady=5)
        ent.insert(0, "Немедленно рассмеши зрителей!")

        lbl = ttk.Label(self.panel, text="")
        lbl.pack(pady=5)

        def send():
            if not self.plugin_queue:
                messagebox.showwarning("Предупреждение", "Пожалуйста, сначала запустите стрим перед нажатием")
                return
            msg = ent.get().strip()
            if not msg:
                return
            self.send_text(self.plugin_queue, f"[Инструкция режиссёра] {msg}")
            lbl.config(text="✅ Отправлено!", foreground="green")
            logger.info(f"[{self.PLUGIN_NAME}] Ручная реплика отправлена: {msg}")

        ttk.Button(self.panel, text="📨 Отправить ИИ", command=send).pack(pady=10)
```

### Паттерн is_connected=True фиксированный (всегда активный тип)

Если плагин не требует внешнего подключения, но нужно чтобы значок всегда был активен независимо от настройки `enabled`,
установите `is_connected = True` как фиксированное значение.

```python
class AlwaysActiveToolPlugin(BasePlugin):
    PLUGIN_ID   = "always_active_plugin"
    PLUGIN_NAME = "🔧 Всегда активный плагин"
    PLUGIN_TYPE = "TOOL"

    def __init__(self):
        super().__init__()
        self.is_connected = True  # Зафиксировано True → значок всегда активен
        self.plugin_queue = None

    def get_default_settings(self):
        return {"enabled": True}

    def start(self, prompt_config, plugin_queue):
        self.plugin_queue = plugin_queue

    def stop(self):
        self.plugin_queue = None
```

---

## 9. Как создать плагин команд CMD

Плагин, выполняющий обработку, когда ИИ пишет `[CMD:XXX] аргумент` в своём выводе.

### Принцип работы

```
Вывод ИИ:  "Давайте отображу изображение [CMD:IMG] создать закатный пейзаж"
              ↓
TeloPon обнаруживает [CMD:IMG]
              ↓
Вызывается handle("создать закатный пейзаж") плагина ImgPlugin
```

### Минимальный пример

```python
from plugin_manager import BasePlugin
import logger


class MyCommandPlugin(BasePlugin):
    IDENTIFIER = "MYCMD"   # ← Реагирует на [CMD:MYCMD]. Не устанавливать PLUGIN_ID
    REQUIRED_CONFIG = []

    def handle(self, value: str):
        """Часть после [CMD:MYCMD] передаётся как value"""
        logger.info(f"[CMD:MYCMD] Получено: {value}")
        # Напишите здесь свою обработку


plugin = MyCommandPlugin()
```

### Механизм автоматической регистрации

CMD плагины **автоматически регистрируются простым размещением `.py` файла в папке `plugins/`**.
При наличии `IDENTIFIER` приложение автоматически сканирует папку при запуске.

```
TeloPon/
 └── plugins/
      └── my_command_plugin.py   ← Размещение здесь активирует [CMD:MYCMD]
```

### Как использовать REQUIRED_CONFIG

Если плагин не может работать без определённых настроек (например, API-ключей), перечислите ключи настроек в `REQUIRED_CONFIG`.
Если они не настроены, плагин будет автоматически пропущен, предотвращая ошибки.

```python
class WeatherPlugin(BasePlugin):
    IDENTIFIER      = "WEATHER"
    REQUIRED_CONFIG = ["weather_api_key"]  # требуется config.weather_api_key

    def handle(self, value: str):
        import config
        api_key = config.app_config["weather_api_key"]
        # ...
```

---

## 10. Гибридный плагин: объединение панели интерфейса и CMD

Установка как `PLUGIN_ID`, так и `IDENTIFIER` даёт плагину функциональность как панели интерфейса, так и команд CMD.

```python
from plugin_manager import BasePlugin
import logger


class HybridPlugin(BasePlugin):
    # Зарегистрирован как панель интерфейса
    PLUGIN_ID   = "hybrid_plugin"
    PLUGIN_NAME = "🔀 Гибридный плагин"
    PLUGIN_TYPE = "TOOL"

    # Также зарегистрирован как команда CMD
    IDENTIFIER = "HYBRID"

    def __init__(self):
        super().__init__()
        self.plugin_queue = None

    # ----- Сторона панели интерфейса -----
    def start(self, prompt_config, plugin_queue):
        self.plugin_queue = plugin_queue

    def stop(self):
        self.plugin_queue = None

    def open_settings_ui(self, parent_window):
        # ... панель интерфейса настроек

    # ----- Сторона команды CMD -----
    def handle(self, value: str):
        """Обработка, вызываемая при получении [CMD:HYBRID]"""
        logger.info(f"[CMD:HYBRID] Получено: {value}")
        if self.plugin_queue:
            self.send_text(self.plugin_queue, f"Гибридная обработка: {value}")
```

Гибридные плагины также автоматически регистрируются простым размещением в папке `plugins/`.

---

## 11. Пример существующего плагина: img_plugin.py

Пример реально используемого CMD плагина (`plugins/img_plugin.py`).

```
[CMD:IMG] создать закатного кота   → Сгенерировать и отобразить изображение с помощью Gemini AI
[CMD:IMG] title                    → Отобразить TITLE.png в большом формате
[CMD:IMG] last                     → Повторно отобразить ранее сгенерированное изображение
[CMD:IMG] hide                     → Скрыть изображение
[CMD:IMG] big                      → Отобразить изображение в большом формате
[CMD:IMG] small                    → Отобразить изображение в уменьшенном размере
```

`ImgPlugin` является только CMD, поэтому у него нет `PLUGIN_ID` (не появляется в панели интерфейса),
и установлен только `IDENTIFIER = "IMG"`.

---

## 11. Сводка часто используемых методов и утилит

### send_text / send_image (встроено в BasePlugin)

```python
# Отправить текст ИИ
self.send_text(self.plugin_queue, "Сообщение для ИИ")

# Отправить изображение ИИ
# send_image(queue, image_bytes, mime_type="image/jpeg")
#   queue       : передать self.plugin_queue
#   image_bytes : данные изображения в формате bytes
#   mime_type   : необязателен. по умолчанию "image/jpeg" (можно передать как позиционный или именованный аргумент)
self.send_image(self.plugin_queue, img_bytes, mime_type="image/jpeg")
```

### logger (стандартный вывод лога TeloPon)

```python
import logger

logger.info("Обычный лог (белый/зелёный)")
logger.warning("Предупреждение (жёлтый)")
logger.error("Ошибка (красный)")
logger.debug("Отладочный лог (приглушённый цвет. Отображается только при запуске с параметром -d)")
```

---

## 13. Контрольный список разработки плагинов

### При создании плагина панели интерфейса

- [ ] Установлен `PLUGIN_ID` (уникальная буквенно-цифровая строка)
- [ ] Установлен `PLUGIN_NAME` (имя, отображаемое в интерфейсе)
- [ ] Установлен `PLUGIN_TYPE` как `"BACKGROUND"` или `"TOOL"`
- [ ] Файл `.py` размещён в папке `plugins/`
- [ ] Если поток запускается в `start()`, флаг устанавливается в False в `stop()`
- [ ] Добавлен `time.sleep()` внутри цикла `while` (забыть об этом приводит к 100% загрузке CPU)
- [ ] Потенциально проблемный код обёрнут в `try-except`
- [ ] Определена стратегия отображения значка (использовать `is_connected` или запасной вариант `enabled`)

### При создании плагина команд CMD

- [ ] Установлен `IDENTIFIER` (рекомендуются заглавные буквы; это XXX в `[CMD:XXX]`)
- [ ] Реализован `handle(self, value: str)`
- [ ] Файл `.py` размещён в папке `plugins/`
- [ ] Если требуемые настройки существуют, имена ключей перечислены в `REQUIRED_CONFIG`

### Общее

- [ ] Импорт с `from plugin_manager import BasePlugin`
- [ ] Вызов `super().__init__()` в начале `__init__` (обязателен при использовании `PLUGIN_ID`)
- [ ] Использование `logger.info` / `logger.warning` для вывода лога

---

## 13. Распространённые ошибки и способы их исправления

**Плагин не появляется в панели интерфейса**
→ Убедитесь, что `PLUGIN_ID` не по-прежнему установлен в `""`. Если пусто, сканирование не выполняется.

**CMD не реагирует**
→ Убедитесь, что `IDENTIFIER` установлен. Также проверьте, что файл `.py` размещён в папке `plugins/`.

**Система зависает во время стриминга**
→ Причина — отсутствующий `time.sleep(1.0)` внутри цикла `while self.is_running:`.

**Поток не останавливается**
→ Убедитесь, что `stop()` устанавливает флаг в `False`. Установка потоков в `daemon=True` является безопасной практикой.

**Настройки не сохраняются**
→ `save_settings()` не будет работать, если `PLUGIN_ID` не установлен. Убедитесь, что вызывается `super().__init__()`.

**enabled=True установлен, но значок не светится**
→ Если в `__init__` определены `self.is_connected = False` или `self.is_running = False`, они имеют приоритет над `enabled` и значок остаётся серым.
Для плагинов без внешнего подключения выберите один из вариантов:
- **Не определять `is_connected` / `is_running`** → используется запасной вариант `enabled` (рекомендуется для простых плагинов)
- **Зафиксировать `self.is_connected = True`** → значок всегда активен (используйте, когда нужно чтобы значок всегда светился)

Инициализация `is_connected = False` с последующим переключением на `True` ведёт себя как плагин с подключением к внешнему сервису (значок серый пока не подключено). При намеренном использовании — это правильный паттерн.

**Настройки `[CMD:IMG]` не применяются**
→ Проверьте, что `img_plugin.py` размещён в папке `plugins/`.
