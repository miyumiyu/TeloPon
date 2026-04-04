# 🎮 VCI OSC телоп (VciOscPlugin.py)

> 📥 **[Скачать плагин](https://github.com/miyumiyu/TeloPon/releases/download/plugins-v1.3/VciOscPlugin.py)**

Этот плагин отправляет текст телопов ИИ в **VirtualCast VCI** в реальном времени по протоколу **OSC (OpenSound Control)**.
Позволяет отображать телопы в виртуальном пространстве VirtualCast.

---

## 🛠️ Подготовка

### Требования

1. **VirtualCast** (PC-версия) установлен и запущен
2. Функция **приёма OSC** включена в настройках VirtualCast (порт: 19100)
3. VCI для отображения телопов размещён в VirtualCast

### Настройка OSC в VirtualCast

1. Титульный экран VirtualCast → настройки **«VCI»**
2. Установите **«Функция приёма OSC»** → **«Включено»** или **«Только для создателей»**
3. Убедитесь, что порт приёма — **19100** (по умолчанию)

### Скрипт приёма VCI (Lua)

```lua
vci.osc.RegisterMethod("/telopon/telop/text", function(sender, name, args)
    local text = args[1]
    vci.assets.SetText("TextBoard", text)
end, {ExportOscType.String})
```

---

## ⚙️ Настройка TeloPon

### 1. Открыть панель управления

В панели расширений нажмите **«Панель управления»** рядом с **«VCI OSC телоп»**.

<img width="400" alt="Настройки VCI OSC" src="../../images/VciOscPlugin.png" />

### 2. Включить отправку OSC

Установите флажок **«Отправка OSC ВКЛ»**.

### 3. Настройки подключения

| Настройка | По умолчанию | Описание |
|---|---|---|
| **IP-адрес** | `127.0.0.1` | IP компьютера с VirtualCast |
| **Порт** | `19100` | Порт приёма OSC VirtualCast |
| **OSC-адрес** | `/telopon/telop/text` | Должен совпадать с `RegisterMethod` в VCI |

### 4. Настройки содержания

| Настройка | По умолчанию | Описание |
|---|---|---|
| **📌 Отправлять TOPIC** | ВКЛ | Формат «TOPIC \| MAIN» |
| **🏷️ Отправлять значок** | ВЫКЛ | Отправляется на `/address/badge` |

---

## 📡 Отправляемые OSC-сообщения

| OSC-адрес | Тип | Содержание |
|---|---|---|
| `/telopon/telop/text` | String | Текст телопа |
| `/telopon/telop/text/badge` | String | Название значка (опционально) |
| `/telopon/telop/text/window` | String | Тип окна |

---
