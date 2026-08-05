# Умный дом: Xiaomi Mi Scale 2 → ESP32 → Алиса

[![Sponsor](https://img.shields.io/badge/Sponsor-💖-ff69b4)](https://github.com/sponsors/anton5714) [![Buy Me a Coffee](https://img.shields.io/badge/Buy%20me%20a%20coffee-☕-ffdd57)](https://buymeacoffee.com/chaplyginai)

ESP32 считывает данные с Xiaomi Mi Scale 2 по Bluetooth и передаёт их в Home Assistant; колонка с Алисой (через интеграцию в Home Assistant) озвучивает показатели и даёт короткие рекомендации.

Цель репозитория — дать понятный, безопасный и полностью настраиваемый эталон конфигурации для домашней интеграции весов, прошивок ESP32, автоматизаций и инструкций по публикации и монетизации проекта.

Состояние: рабочая базовая реализация, готова к расширению.

Содержание (быстро)
- Quick Start — пошагово, чтобы подключить весы и получить озвучивание
- Что в репозитории — где какие файлы
- Secrets и безопасность
- Прошивка ESP32 (ESPHome) — как прошить и настроить
- Шаблонные сенсоры и helpers — как подключить в Home Assistant
- Автоматизации — как включить и протестировать
- Донаты — Buy Me a Coffee + USDT (TRC20)

---

Quick Start — подключаем Xiaomi Mi Scale 2 к Home Assistant (пошагово)

1) Что нужно
- ESP32 (например, ESP32‑DevKitC, WROOM)
- Xiaomi Mi Scale 2
- Home Assistant (локально/на сервере)
- Компьютер с USB для прошивки или доступ к ESPHome add-on в Home Assistant
- Аккаунт Buy Me a Coffee (если хотите принять донаты) — https://buymeacoffee.com/chaplyginai
- (Опционально) crypto: USDT (TRC20): TTEGWGEhATMxwJj1VZdnYYBapBuK3T3Yjp

2) Клонируем репозиторий

```bash
git clone https://github.com/anton5714/Home-assistant-smart-home-scale.git
cd Home-assistant-smart-home-scale
```

3) Подготовьте secrets (локально в вашем Home Assistant)
- В корне конфигурации HA создайте или отредактируйте `secrets.yaml` (не пушьте в репо):

```yaml
wifi_ssid: "ВАША_WIFI"
wifi_password: "ВАШ_WIFI_PASSWORD"
esphome_api_encryption_key: "случайная_строка"
esphome_ota_password: "случайная_строка"
```

4) Прошивка ESP32 (ESPHome)
- Откройте `firmware/esp32_wroom_scale.yaml` — в нём уже используются `!secret` для приватных данных.
- Если у вас локально установлен ESPHome (CLI) — выполните:

```bash
esphome run firmware/esp32_wroom_scale.yaml
```

- Или используйте аддон ESPHome в Home Assistant: загрузите конфиг, скомпилируйте и прошейте по USB.

5) Настройка BLE парсинга (варианты)
Вариант A — использовать BLE Monitor (рекомендуется, если не хотите писать C++):
- Установите HACS и затем интеграцию BLE Monitor.
- Подключите ESPHome как `bluetooth_proxy` (в `esp32_wroom_scale.yaml` у вас включён bluetooth_proxy).
- В настройках BLE Monitor добавьте устройство Mi Scale (по MAC или по имени) и включите нужные сенсоры (вес, impedance). BLE Monitor будет автоматически создавать сенсоры в Home Assistant.

Вариант B — парсить на ESPHome (сложнее):
- Требуется кастомный компонент в ESPHome или сторонняя библиотека, чтобы распарсить рекламные пакеты Mi Scale и сформировать sensora `weight`, `impedance`.
- Если хотите — я подготовлю пример кастомного C++ блока для esp32 — скажите "парсер".

6) Подключаем шаблонные сенсоры в Home Assistant
- Скопируйте `config/sensors_body.yaml` и `config/helpers_body.yaml` в папку конфигурации Home Assistant (обычно `/config/`).
- В `configuration.yaml` добавьте/проверьте строки:

```yaml
template: !include config/sensors_body.yaml
input_number: !include config/helpers_body.yaml
```

- Перезагрузите Home Assistant и заполните helpers (рост, возраст, пол) в UI: Settings → Devices & Services → Helpers.

7) Подключаем автоматизации (озвучивание)
- Откройте `automations/announce_mi_scale.yaml` и `automations/announce_mi_scale_debounce.yaml`.
- Отредактируйте `device_id` → замените на ваш `entity_id: media_player.имя_плеера` для удобства.
- Импортируйте YAML через UI: Configuration → Automations → Add Automation → YAML или поместите файлы в папку `automations/` конфигурации и Reload Automations.

8) Тестирование
- Поставьте весы на ровную поверхность и встаньте на них.
- В Home Assistant в Developer Tools → States проверьте появление `sensor.vesy_weight` (или sensor, который создаёт BLE Monitor).
- Убедитесь, что автоматизация сработала: проверьте журнал (Configuration → Logs) и устройство `media_player` — должно воспроизвести TTS/команду.

9) Отладка — частые проблемы
- Вес не появляется: проверьте, что ESP32 видит рекламные пакеты (лог ESPHome) и bluetooth_proxy активен.
- Некорректные числа: BLE-пакеты иногда содержат запятую — в шаблонах мы делаем replace(',', '.') для безопасного парсинга.
- Повторные срабатывания: используйте debounce-версию автоматизации или добавьте условие (change > 0.1 кг).

10) Дальше — улучшения
- Сделать хранение истории весов (Recorder/History) и графики.  
- Добавить страницы в GitHub Pages: Settings → Pages → main → /docs.  
- Добавить парсер Mi Scale в firmware (я могу помочь).

---

Если нужна помощь на конкретном шаге (например, HACS + BLE Monitor, или подготовка кастомного ESPHome-кода для парсинга Mi Scale) — скажите, и я сгенерирую детальную инструкцию и файлы для коммита.