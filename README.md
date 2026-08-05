# Умный дом: Xiaomi Mi Scale 2 → ESP32 → Алиса

[![Sponsor](https://img.shields.io/badge/Sponsor-💖-ff69b4)](https://github.com/sponsors/anton5714) [![Buy Me a Coffee](https://img.shields.io/badge/Buy%20me%20a%20coffee-☕-ffdd57)](https://buymeacoffee.com/anton5714)

ESP32 считывает данные с Xiaomi Mi Scale 2 по Bluetooth и передаёт их в Home Assistant; колонка с Алисой (через интеграцию в Home Assistant) озвучивает показатели и даёт короткие рекомендации.

Цель репозитория — дать понятный, безопасный и полностью настраиваемый эталон конфигурации для домашней интеграции весов, прошивок ESP32, автоматизаций и инструкций по публикации и монетизации проекта.

Состояние: рабочая базовая реализация, готова к расширению.

Table of Contents
- Quick Start
- Что в репозитории
- Secrets и безопасность
- Прошивка ESP32 (ESPHome)
- Шаблонные сенсоры и helpers
- Автоматизации (озвучивание и ИИ-совет)
- GitHub Pages (документация)
- Как помочь / Донаты
- Contributing

Quick Start
1. Клонируйте репозиторий:

   git clone https://github.com/anton5714/Home-assistant-smart-home-scale.git
   cd Home-assistant-smart-home-scale

2. Добавьте секреты в `secrets.yaml` (локально, не пушьте):

   ```yaml
   wifi_ssid: "ваша_wifi_сеть"
   wifi_password: "ваш_wifi_пароль"
   esphome_api_encryption_key: "случайная_строка_для_API_шифрования"
   esphome_ota_password: "случайная_строка_для_OTA"
   ```

   Генерация безопасных строк:
   - head -c 32 /dev/urandom | base64
   - openssl rand -base64 18

3. Прошивка ESP32 (ESPHome):
   - Откройте `firmware/esp32_wroom_scale.yaml` и замените секреты через `secrets.yaml`.
   - Используйте ESPHome (аддон в Home Assistant или esphome CLI):
     esphome run firmware/esp32_wroom_scale.yaml

4. Подключите шаблонные сенсоры и helpers в Home Assistant:
   - Поместите `config/sensors_body.yaml` и `config/helpers_body.yaml` в `config/` вашей конфигурации HA.
   - В `configuration.yaml` добавьте:
     ```yaml
     template: !include config/sensors_body.yaml
     input_number: !include config/helpers_body.yaml
     ```
   - Перезагрузите HA и заполните helpers (рост/возраст/пол) через UI.

5. Импортируйте автоматизации:
   - Скопируйте содержимое `automations/announce_mi_scale.yaml` и `automations/announce_mi_scale_debounce.yaml` в вашу конфигурацию автоматизаций (или через UI → YAML import).
   - Отредактируйте `device_id` или замените на `entity_id: media_player.your_player` и проверьте имена сенсоров.

6. Документация: откройте `docs/` для подробных инструкций и отладки.

Что в репозитории
- README.md — этот файл
- automations/ — YAML автоматизаций для Home Assistant (рабочая и debounce версии)
- config/ — шаблонные сенсоры для анализа тела (`sensors_body.yaml`, `sensors_body_improved.yaml`) и helpers (`helpers_body.yaml`)
- firmware/ — пример ESPHome-конфига `esp32_wroom_scale.yaml` (все секреты = !secret)
- docs/ — дополнительные инструкции (`mi-scale-setup.md`, `firmware-secrets.md`, `sensors-setup.md`, `buymeacoffee-setup.md`)
- DONATE.md — информация о том, как поддержать проект (включая крипто адреса)
- LICENSE — MIT

Secrets и безопасность
- Никогда не пушьте реальных паролей и ключей. Добавьте `secrets.yaml` в `.gitignore`.
- Все публичные конфиги в `firmware/` используют `!secret`.
- Если ранее вы публиковали ключи — смените их (ротация).

Криптовалюта (USDT TRC20 / Tron)
USDT (TRC20 / Tron): TTEGWGEhATMxwJj1VZdnYYBapBuK3T3Yjp

Пожалуйста, убедитесь, что отправляете токены именно в сети TRON (TRC20), иначе средства могут быть потеряны.

Прошивка ESP32 (коротко)
- Рекомендуется ESPHome (удобно взаимодействует с Home Assistant).
- Конфиг `firmware/esp32_wroom_scale.yaml` включает `esp32_ble_tracker` и `bluetooth_proxy` — это позволяет сканировать BLE и пробрасывать данные в Home Assistant.
- Для реального парсинга Mi Scale может потребоваться кастомный компонент или парсинг на стороне Home Assistant (например, BLE Monitor). См. docs/.

Шаблонные сенсоры и helpers
- `config/sensors_body.yaml` — базовый набор сенсоров (жир, мышцы, ВБ, BMI и т.д.).
- `config/sensors_body_improved.yaml` — улучшённые формулы (BMR по Mifflin–St Jeor, BMI с ростом из input_number).
- `config/helpers_body.yaml` — UI-помощники: рост, возраст, пол.

Автоматизации
- `automations/announce_mi_scale.yaml` — ваша рабочая автоматизация: пауза, озвучивание, отправка текста-команды для получения ИИ-совета.
- `automations/announce_mi_scale_debounce.yaml` — версия с debounce (условие изменения >0.1 кг и for: 3s).

GitHub Pages
- Вы можете опубликовать документацию: Settings → Pages → Source → main / (root or /docs) → Save.
- Рекомендую выбрать `docs/` как источник и включить Pages — тогда сайт будет доступен по https://anton5714.github.io/Home-assistant-smart-home-scale/

Как помочь / Донаты
Если вам понравился проект и вы хотите поддержать развитие — спасибо! Поддержка помогает покупать оборудование, тестировать новые сценарии и писать подробные инструкции.

Поддержать проект:
- GitHub Sponsors: https://github.com/sponsors/anton5714
- Buy Me a Coffee: https://buymeacoffee.com/anton5714
- Ko-fi: https://ko-fi.com/ВАШ_АККАУНТ
- Patreon: https://patreon.com/ВАШ_АККАУНТ
- PayPal / YooMoney: [ваша ссылка]

Contributing
- Открывайте issues с подробным описанием проблемы/фичи.
- PR: форк → ветка → pull request. Используйте осмысленные сообщения коммитов (feat:, fix:, docs:, chore:).
- Код и конфиги: добавляйте комментарии и документацию.

Лицензия
Проект распространяется под MIT License. См. LICENSE.
