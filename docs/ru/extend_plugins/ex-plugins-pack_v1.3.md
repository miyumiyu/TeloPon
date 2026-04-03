[English](https://github.com/miyumiyu/TeloPon/blob/main/docs/en/extend_plugins/ex-plugins-pack_v1.3.md) | [日本語](https://github.com/miyumiyu/TeloPon/blob/main/docs/ja/extend_plugins/ex-plugins-pack_v1.3.md) | [한국어](https://github.com/miyumiyu/TeloPon/blob/main/docs/ko/extend_plugins/ex-plugins-pack_v1.3.md) | **Русский**

Новейшая версия официального пакета расширений для TeloPon!
Скачайте только нужные файлы (`.py`) и добавьте их в свой TeloPon.
**※ Доступно начиная с версии V2.14b.**

## 📦 Включённые плагины

### 🆕 🔊 VOICEVOX TTS (`voicevox_plugin.py`)

![Настройки VOICEVOX TTS](https://github.com/miyumiyu/TeloPon/blob/main/images/voicevox_plugin.png?raw=true)

Озвучивает текст телопов, сгенерированных ИИ, в реальном времени с помощью движка синтеза речи **VOICEVOX**.
* **Разнообразие голосов**: поддержка всех персонажей VOICEVOX + стилей (характеров). Простой выбор через выпадающий список.
* **Отображение иконки**: иконка выбранного персонажа показана в панели настроек.
* **Тонкая настройка**: 5 параметров — скорость, громкость, высота тона, интонация и тишина перед речью — с мгновенной настройкой.
* **Чтение TOPIC**: при включении читает верхнюю строку текста (TOPIC) вместе с основным.
* **Задержка телопа**: синхронизация озвучки с появлением телопа в OBS.
* **Автопроверка подключения**: проверяет соединение с VOICEVOX при запуске. Автоматически отключается, если недоступен.

📥 [Скачать плагин](https://github.com/miyumiyu/TeloPon/releases/download/plugins-v1.3/voicevox_plugin.py)
👉 [Подробное руководство](../plugins/voicevox_plugin.md)

> 💡 **Требования**: установите [VOICEVOX](https://voicevox.hiroshiba.jp/) и запустите одновременно с TeloPon.

### 💬 Discord — интеграция в реальном времени (`discord_integration.py`)
Получает комментарии из указанного канала Discord-сервера в реальном времени и передаёт их ИИ.
* **Сверхпростая настройка**: введите токен бота и нажмите кнопку — ссылка-приглашение генерируется автоматически.
* **Полностью реальное время**: мгновенный сбор комментариев без задержки.

<img width="400" alt="image" src="https://github.com/user-attachments/assets/c8a8585c-52ca-43eb-9cdb-fede4373262a" />

📥 [Скачать плагин](https://github.com/miyumiyu/TeloPon/releases/download/plugins-v1.3/discord_integration.py)
👉 [Подробное руководство](../plugins/discord_integration.md)

### 🏢 Slack — интеграция комментариев (`slack_integration.py`)
Получает комментарии из указанного канала Slack в реальном времени и передаёт их ИИ.
* **Поддержка Socket Mode**: использует новейший способ связи, применяемый в корпоративной среде, для полностью реального приёма без задержки.
* **Автоматическое преобразование имён**: автоматически преобразует алфавитно-цифровые ID пользователей Slack в отображаемые имена.

<img width="400" alt="image" src="https://github.com/user-attachments/assets/1f228c32-3552-453e-aa29-c824da5b6795" />

📥 [Скачать плагин](https://github.com/miyumiyu/TeloPon/releases/download/plugins-v1.3/slack_integration.py)
👉 [Подробное руководство](../plugins/slack_integration.md)

---

## 🛠️ Как установить плагины

1. Нажмите на нужный файл (`.py`) в разделе **Assets** внизу этой страницы, чтобы скачать его.
2. Откройте папку `TeloPon-XXX` на вашем компьютере.
3. Поместите скачанный файл `.py` в папку **`plugins`**.
4. Запустите TeloPon — новая функция появится в списке «🔌 Расширения» справа!

※ Подробные инструкции по настройке см. на экране настроек каждого плагина (⚙️) или в официальном руководстве.

---

## 📋 Название релиза (для GitHub Releases)

```
🔌 TeloPon Официальный пакет расширений v1.3 (VOICEVOX, Discord и Slack)
```
