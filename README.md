# Home Assistant — Mi Scale
Цель: максимально простая, рабочая и безопасная структура — только нужные файлы, чтобы любой пользователь мог быстро подключить Xiaomi Mi Scale 2 к Home Assistant через ESP32 и получить два основных сенсора: вес и импеданс, а также шаблонные датчики (ИМТ, пример %жира) и простую автоматизацию для озвучивания через Алису.

Вот так это работает: https://youtube.com/shorts/LJl9GLgwjTk?feature=share

Всё, что нужно сделать (3 шага)

1) Прошить ESP32
- Откройте `esphome/esp-bt.yaml` и заполните `!secret wifi_ssid`/`!secret wifi_password` в своём `secrets.yaml` (см. `secrets.yaml.example`).
- Если ваш ESPHome поддерживает `xiaomi_miscale2`, вставьте MAC ваших весов в блок `sensor: platform: xiaomi_miscale2` (см. комментарий в файле). Иначе используйте `esp32_ble_tracker`+`bluetooth_proxy` и парсер на стороне Home Assistant (BLE Monitor).
- Прошивка:
  ```bash
  esphome run esphome/esp-bt.yaml
  ```

2) Добавить шаблонные датчики в Home Assistant
- Скопируйте файл `home-assistant/sensors/body_analysis.yaml` в вашу конфигурацию (например в `/config/home-assistant/sensors/body_analysis.yaml`) и подключите его в `configuration.yaml`:
  ```yaml
  template: !include home-assistant/sensors/body_analysis.yaml
  ```
- Создайте helpers (через UI):
  - input_number.user_height (рост в см)
  - input_number.user_age (возраст)
  - input_select.user_gender (male / female)
- После этого у вас появятся шаблонные сенсоры: `sensor.body_bmi`, `sensor.body_fat_percentage`, `sensor.muscle_mass`.

3) Добавить автоматизацию (озвучивание через Алису)
- Положите `home-assistant/automations/alice_announce.yaml` в папку `automations/` (или импортируйте через UI).
- В `secrets.yaml` укажите `alice_media_player_entity_id` (например `media_player.yandex_station`).
- Если вы используете навык Алисы с командой `sendText`, настройте rest_command (пример внизу README) — иначе автоматизация будет использовать TTS через ваш media_player.

Примечания и безопасность
- Не публикуйте `secrets.yaml` в публичных репозиториях. Используйте `secrets.yaml.example` как шаблон.
- В репозитории все чувствительные значения заменены на `!secret` или placeholders.

---

Файлы в репозитории (итог)
- esphome/esp-bt.yaml — ESPHome конфиг (sanitized, с комментарием куда вставить MAC)
- home-assistant/sensors/body_analysis.yaml — шаблонные template sensors (BMI, fat%, muscle)
- home-assistant/automations/alice_announce.yaml — простая автоматизация озвучивания
- secrets.yaml.example — пример секретов
- .gitignore — игнор secrets.yaml
- README.md — вы сейчас читаете минимальную one‑page инструкцию
- LICENSE — MIT

Если нужно дополнительное упрощение — скажите, но перед ��юбыми удалениями или изменениями я буду просить подтверждение.
