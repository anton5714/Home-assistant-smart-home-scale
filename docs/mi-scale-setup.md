# Инструкция: подключение Xiaomi Mi Scale 2 к ESP32 и Home Assistant (пошагово)

Эта страница подробно объясняет, как получить рабочие сенсоры веса и интегрировать их в Home Assistant.

1) Требования
- ESP32 с ESPHome (см. firmware/esp32_wroom_scale.yaml)
- Home Assistant с доступом к HACS (рекомендуется)
- Xiaomi Mi Scale 2 рядом с ESP32 (в зоне действия Bluetooth)

2) Подготовка ESPHome
- В `firmware/esp32_wroom_scale.yaml` включены:
  - `esp32_ble_tracker` — сканирование BLE
  - `bluetooth_proxy` — проброс BLE в Home Assistant
- Убедитесь, что в `secrets.yaml` добавлены wifi и пароли (не пушьте secrets в репо).

3) Вариант: распознавание Mi Scale через BLE Monitor (рекомендуемый)
- Установите HACS (если ещё не установлен) и установите интеграцию "BLE Monitor".
- В настройках BLE Monitor создайте запись для Mi Scale:
  - device: укажите MAC-адрес весов (можно найти в логах ESPHome или в Developer Tools → States при включённом bluetooth_proxy)
  - выберите, какие параметры парсить: weight, impedance, battery и т.д.
- BLE Monitor создаст sensora в Home Assistant: `sensor.mi_scale_weight`, `sensor.mi_scale_impedance` и т.д.

Пример конфигурации BLE Monitor через UI — достаточно указать MAC и тип девайса.

4) Вариант: парсинг в ESPHome (если вы хотите, чтобы ESP отправлял готовые sensora в HA)
- Потребуется кастомный компонент (C++ код) или использование готовых community-компонентов. Это сложнее, но даёт более автономную систему.
- Если нужно, я могу подготовить пример кастомного кода и интеграции для ESPHome.

5) Подключение шаблонных сенсоров (расчёты)
- После появления базовых sensora weight/impedance подключите шаблонные сенсоры из `config/sensors_body.yaml` и `config/sensors_body_improved.yaml`.
- Пример включения в `configuration.yaml`:

```yaml
template: !include config/sensors_body.yaml
input_number: !include config/helpers_body.yaml
```

- Заполните `input_number.user_height` и `input_number.user_age` в UI → Helpers.

6) Автоматизации (озвучивание)
- Используйте `automations/announce_mi_scale.yaml` (или debounce-версию) и укажите `entity_id` вашего media_player.
- Пример правки (замените device_id на entity_id):

```yaml
action:
  - service: media_player.play_media
    target:
      entity_id: media_player.yandex_station
    data:
      media_content_type: text
      media_content_id: "Текст для озвучивания"
```

7) Тестирование и отладка
- Посмотрите логи ESPHome (`esphome logs firmware/esp32_wroom_scale.yaml`) — там увидите обнаружение Mi Scale.
- В Home Assistant проверьте состояние sensor.* и запустите автоматизацию вручную (Developer Tools → Services → automation.trigger).
- Если звук не проигрывается, проверьте, что media_player поддерживает TTS/команды и что у него есть подключение к сети.

8) Частые ошибки и решения
- ESP не видит весы: проверьте расстояние/питание весов и перезапустите ESP.
- Нет импеданса или неверные значения: используйте BLE Monitor (там чаще легче получить корректные поля).
- Автоматизация слишком часто срабатывает: используйте `for:` в триггере или условие изменения > 0.1.

9) Если хотите, я могу:
- Подготовить HACS + BLE Monitor конфиг шаг за шагом (с примерами UI полей).  
- Сгенерировать кастомный ESPHome-код для парсинга рекламных пакетов Mi Scale и отправки sensora в HA.  

---

Если вы готовы — скажите, какой вариант парсинга предпочитаете: "BLE Monitor (рекомендую)" или "ESPHome parser (хочу пример кастомного компонента)", и я подготовлю подробное руководство и необходимые файлы.