# Home Assistant — Mi Scale (one‑page)

Краткая, пошаговая инструкция — всё на одной странице: как прошить ESP32, подключить Xiaomi Mi Scale 2 по BLE и настроить автоматизацию с озвучиванием через Алису.

Требуется
- Xiaomi Mi Scale 2 (BLE)
- ESP32 (ESP32-WROOM / DevKitC)
- Home Assistant (локальная инстанция)
- ESPHome (аддон или CLI)
- (Опционально) HACS + BLE Monitor / bodymiscale для расширенных вычислений

1) Подготовка секретов
- Скопируйте `secrets.yaml.example` в `secrets.yaml` и заполните значения локально (не пушьте в репо).
  - `wifi_ssid`, `wifi_password` — ваши Wi‑Fi данные
  - `esphome_api_encryption_key`, `esphome_ota_password` — безопасные строки
  - `alice_media_player_entity_id` — entity_id медиаплеера (например `media_player.yandex_station`)

2) Прошивка ESP32 (ESPHome)
- Подключите ESP32 к компьютеру или используйте ESPHome аддон.
- Прошивка (локально):
  ```bash
  esphome run esphome/esp-bt.yaml
  ```
- При первом запуске ESP создаст временную точку доступа, если не сможет подключиться к Wi‑Fi.

3) Найти MAC весов
- Пробудите весы (коснитесь или встаньте на них).
- Мобильный: nRF Connect → Start Scan → найдите устройство с именем `MIBCS`/`Mi` → запишите MAC.
- Linux: `sudo hcitool lescan --duplicates` → пробудите весы → запишите MAC.
- После первой прошивки посмотрите логи ESPHome: `esphome logs esphome/esp-bt.yaml` — там ESP показывает обнаруженные MAC.

4) Конфигурация: два варианта
- Вариант A — если ESPHome поддерживает `xiaomi_miscale2`:
  - Добавьте в `esphome/esp-bt.yaml` блок `sensor: platform: xiaomi_miscale2` с вашим MAC и прошейте.
  - В Home Assistant появятся `sensor.mi_scale_weight` и `sensor.mi_scale_impedance`.
- Вариант B — универсальный: включите `esp32_ble_tracker` + `bluetooth_proxy` в `esphome/esp-bt.yaml`, прошейте, затем установите HACS → BLE Monitor и добавьте устройство по MAC. BLE Monitor создаст `sensor.*`.

5) Подключение шаблонных сенсоров
- Скопируйте `home-assistant/sensors/body_analysis.yaml` в вашу конфигурацию и подключите в `configuration.yaml`:
  ```yaml
  template: !include home-assistant/sensors/body_analysis.yaml
  ```
- В UI создайте/helpers: `input_number.user_height`, `input_number.user_age`, `input_select.user_gender`.

6) Автоматизация: озвучивание + отправка текста в навык Алисы
- В репозитории есть готовая автоматизация: `home-assistant/automations/body_announcement.yaml`.
- Заполните `secrets.yaml` ключом `alice_media_player_entity_id` или замените прямо `entity_id` в YAML.
- Пример последовательности: при изменении веса (stable for 3s) и изменении >0.1 кг — воспроизвести TTS (сообщение с весом и BMI), затем через паузу отправить REST/команду `sendText` в навык Алисы.
- Для работы `sendText` настройте `rest_command.send_text_to_alice` в `configuration.yaml` согласно вашей интеграции с навыком.

7) Тестирование
- Встаньте на весы и дождитесь стабилизации (индикатор перестал мигать).
- В Home Assistant: Developer Tools → States → найдите `sensor.mi_scale_weight`.
- Вручную запустите автоматизацию: Developer Tools → Services → automation.trigger.

8) Отладка (коротко)
- Весы не видны: пробудите физически; убедитесь, что ESP и весы в зоне действия BLE.
- ESP не видит: `esphome logs esphome/esp-bt.yaml`.
- Сенсоры не появились: проверьте Integrations → BLE Monitor / Developer Tools → States.
- Автоматизация срабатывает часто: используйте `for: '00:00:03'` и условие delta > 0.1 кг.

9) Команды
```bash
sudo hcitool lescan --duplicates
esphome run esphome/esp-bt.yaml
esphome logs esphome/esp-bt.yaml
```

---

Если у вас другие имена сенсоров (например `sensor.vesy_weight`), замените их в `home-assistant/sensors/body_analysis.yaml` и в автоматизации на ваши имена (Developer Tools → States покажет текущие entity_id).
