# Краткое руководство и ссылки

Этот репозиторий содержит готовые конфиги и инструкции, чтобы подключить Xiaomi Mi Scale 2 к Home Assistant через ESP32 (ESPHome + BLE proxy).

Необходимое оборудование и ПО
- Xiaomi Mi Scale 2 (BLE)
- ESP32 (на базе ESP32-WROOM / DevKitC)
- Home Assistant (локальная инстанция)
- ESPHome (аддон или CLI)
- HACS (для BLE Monitor / bodymiscale, опционально)

Краткая инструкция (Quick Start)
1. Скопируйте `secrets.yaml.example` в `secrets.yaml` и заполните реальные значения локально (не пушьте).
2. Подключите ESP32 и прошейте `esphome/esp-bt.yaml` через ESPHome (esphome run esphome/esp-bt.yaml).
3. Найдите MAC ваших весов (nRF Connect или `sudo hcitool lescan --duplicates`) и, если используете компонент xiaomi_miscale2, впишите MAC в конфиг и перепрошивайте.
4. Если используете bluetooth_proxy — установите HACS → BLE Monitor и добавьте устройство по MAC; сенсоры появятся в Home Assistant.
5. Подключите шаблонные сенсоры: поместите `home-assistant/sensors/body_analysis.yaml` в вашу конфигурацию и подключите через `template: !include home-assistant/sensors/body_analysis.yaml`.
6. Поместите автоматизацию `home-assistant/automations/body_announcement.yaml` в папку `automations/` и замените media_player на ваш `entity_id` (или используйте secrets).

Как адаптировать под себя
- Имя пользователя, цели и медиаплеер задавайте в `secrets.yaml` и в helpers Home Assistant (input_number / input_select).  
- Шаблоны и пороги (дельта веса для срабатывания, время стабилизации) находятся в `home-assistant/automations/body_announcement.yaml` и `home-assistant/sensors/body_analysis.yaml`.

Дисклеймер
Формулы для расчёта состава тела — приближённые оценки и не являются медицинским диагнозом. Для точных измерений используйте сертифицированное медицинское оборудование.

Список ключевых файлов
- esphome/esp-bt.yaml — конфиг ESPHome (использует !secret для приватных данных)
- home-assistant/sensors/body_analysis.yaml — шаблонные sensors (BMI, fat % пример)
- home-assistant/automations/body_announcement.yaml — автоматизация озвучки и отправки запроса в навык Алисы
- secrets.yaml.example — шаблон для локального secrets.yaml
- DONATE.md — информация о поддержке проекта

Лицензия: MIT (см. LICENSE)
