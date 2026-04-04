[English](https://github.com/miyumiyu/TeloPon/blob/main/docs/en/extend_plugins/ex-plugins-pack_v1.4.md) | [日本語](https://github.com/miyumiyu/TeloPon/blob/main/docs/ja/extend_plugins/ex-plugins-pack_v1.4.md) | [한국어](https://github.com/miyumiyu/TeloPon/blob/main/docs/ko/extend_plugins/ex-plugins-pack_v1.4.md) | **Русский**

Новейшая версия официального пакета расширений для TeloPon!
Скачайте только нужные файлы (`.py`) и добавьте их в свой TeloPon.
**※ Доступно начиная с версии V2.15b.**

## 📦 Включённые плагины

### 🆕 🎮 VCI OSC телоп (`VciOscPlugin.py`)

<img width="400" alt="Настройки VCI OSC" src="https://github.com/miyumiyu/TeloPon/blob/main/images/VciOscPlugin.png?raw=true" />

Отправляет текст телопов ИИ в **VirtualCast VCI** в реальном времени по протоколу **OSC (OpenSound Control)**.
* **Интеграция с VirtualCast**: отображение телопов в виртуальном пространстве
* **Настраиваемый адрес**: IP, порт и OSC-адрес настраиваются
* **TOPIC и значок**: отправка темы и значка (тега эмоции) на отдельные OSC-адреса
* **Тестовая отправка**: проверка соединения одним нажатием

📥 [Скачать плагин](https://github.com/miyumiyu/TeloPon/releases/download/plugins-v1.4/VciOscPlugin.py)  
👉 [Подробное руководство](../plugins/VciOscPlugin.md)

> 💡 **Требования**: включите приём OSC в [VirtualCast](https://virtualcast.jp/) (PC-версия).

### 🔊 VOICEVOX TTS (`voicevox_plugin.py`)

<img width="400" alt="Настройки VOICEVOX TTS" src="https://github.com/miyumiyu/TeloPon/blob/main/images/voicevox_plugin.png?raw=true" />

Озвучивает текст телопов ИИ в реальном времени с помощью **VOICEVOX**.
* **Разнообразие голосов**: все персонажи + стили. Простой выбор.
* **Иконка**: иконка выбранного персонажа в панели настроек.
* **Тонкая настройка**: 5 параметров с мгновенной настройкой.
* **Чтение TOPIC**: при включении читает тему вместе с основным текстом.
* **Задержка телопа**: синхронизация озвучки с появлением в OBS.
* **Автопроверка**: автоматически отключается, если VOICEVOX недоступен.

📥 [Скачать плагин](https://github.com/miyumiyu/TeloPon/releases/download/plugins-v1.4/voicevox_plugin.py)  
👉 [Подробное руководство](../plugins/voicevox_plugin.md)

> 💡 **Требования**: установите [VOICEVOX](https://voicevox.hiroshiba.jp/) и запустите вместе с TeloPon.

### 💬 Discord — интеграция в реальном времени (`discord_integration.py`)
Получает комментарии из канала Discord в реальном времени и передаёт их ИИ.
* **Сверхпростая настройка**: токен бота → кнопка → ссылка-приглашение.
* **Полностью реальное время**: мгновенный сбор без задержки.
* **Поддержка аватаров**: иконки авторов комментариев на телопах reply.

<img width="400" alt="image" src="https://github.com/user-attachments/assets/c8a8585c-52ca-43eb-9cdb-fede4373262a" />

📥 [Скачать плагин](https://github.com/miyumiyu/TeloPon/releases/download/plugins-v1.4/discord_integration.py)  
👉 [Подробное руководство](../plugins/discord_integration.md)

### 🏢 Slack — интеграция комментариев (`slack_integration.py`)
Получает комментарии из канала Slack в реальном времени и передаёт их ИИ.
* **Socket Mode**: приём без задержки.
* **Автопреобразование имён**: ID пользователей → отображаемые имена.
* **Поддержка аватаров**: иконки авторов комментариев на телопах reply.

<img width="400" alt="image" src="https://github.com/user-attachments/assets/1f228c32-3552-453e-aa29-c824da5b6795" />

📥 [Скачать плагин](https://github.com/miyumiyu/TeloPon/releases/download/plugins-v1.4/slack_integration.py)  
👉 [Подробное руководство](../plugins/slack_integration.md)

---

## 🛠️ Как установить плагины

1. Скачайте нужный файл (`.py`) из раздела **Assets** внизу страницы.
2. Откройте папку `TeloPon-XXX` на компьютере.
3. Поместите файл `.py` в папку **`plugins`**.
4. Запустите TeloPon — новая функция появится в списке «🔌 Расширения»!

---

## 📋 Название релиза (для GitHub Releases)

```
🔌 TeloPon Официальный пакет расширений v1.4 (VCI OSC, VOICEVOX, Discord и Slack)
```
