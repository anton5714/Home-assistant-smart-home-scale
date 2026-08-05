# Подключение Xiaomi Mi Scale 2 к Home Assistant через ESP32 (Bluetooth)

Это инструкция содержит чёткие практические шаги для интеграции Xiaomi Mi Scale 2 в Home Assistant с использованием ESP32 и ESPHome.

Требуется
- ESP32 (например, WROOM / DevKitC)
- Xiaomi Mi Scale 2
- Home Assistant (локальная инстанция)
- Компьютер для прошивки ESP32 или ESPHome аддон в Home Assistant
- Доступ к `secrets.yaml` (локально, не храните его в репо)

1) Подготовка файлов
- Клонируйте репозиторий (опционально):
  ```bash
  git clone https://github.com/anton5714/Home-assistant-smart-home-scale.git
  cd Home-assistant-smart-home-scale
  ```
- Отредактируйте локально `/config/secrets.yaml` и добавьте:
  ```yaml
  wifi_ssid: "ВАША_WIFI"
  wifi_password: "ВАШ_WIFI_PASSWORD"
  esphome_api_encryption_key: "случайная_строка"
  esphome_ota_password: "случайная_строка"
  ```

2) Прошивка ESP32 (ESPHome)
- В репозитории есть файл `firmware/esp32_wroom_scale.yaml` с конфигурацией для ESP32. Он использует `!secret` для приватных данных.
- Подключите ESP32 к компьютеру или используйте ESPHome аддон в Home Assistant.
- Выполните (локально с установленным esphome):
  ```bash
  esphome run firmware/esp32_wroom_scale.yaml
  ```
- При первом запуске ESPHome попытается подключиться к Wi‑Fi; если сеть недоступна, ESP создаст временную Wi‑Fi точку (Hotspot) для настройки.

3) Поиск MAC‑адреса весов
- Пробудите весы: коснитесь или встаньте на них (весы начнут вещать BLE).
- Вариант A (мобильный): используйте приложение nRF Connect (Android/iOS) → Start Scan → найдите устройство с именем типа `MIBCS`/`Mi` и скопируйте MAC.
- Вариант B (Linux): в терминале:
  ```bash
  sudo hcitool lescan --duplicates
  ```
  затем пробудите весы и запишите MAC (формат: AA:BB:CC:DD:EE:FF).
- Также можно посмотреть логи ESPHome после первой прошивки — ESP будет сканировать BLE и выведет обнаруженные устройства и MAC в логах.

4) Конфигурация ESPHome для Mi Scale 2
Есть два надёжных варианта:

4A) Если в вашей версии ESPHome есть компонент `xiaomi_miscale2` (простая установка):
- Добавьте в ESP конфиг следующий блок (замените MAC):
  ```yaml
  esp32_ble_tracker:

  bluetooth_proxy:
    active: true

  sensor:
    - platform: xiaomi_miscale2
      mac_address: 'AA:BB:CC:DD:EE:FF'
      name: "Mi Scale 2"
      update_interval: 10s
      weight:
        name: "Mi Scale Weight"
      impedance:
        name: "Mi Scale Impedance"
  ```
- Прошейте ESP ещё раз: `esphome run firmware/esp32_wroom_scale.yaml`.
- В Home Assistant появятся сенсоры веса и импеданса.

4B) Если `xiaomi_miscale2` недоступен: использовать `esp32_ble_tracker` + `bluetooth_proxy` и парсить данные в Home Assistant
- В ESP конфиге включите:
  ```yaml
  esp32_ble_tracker:

  bluetooth_proxy:
    active: true
  ```
- Прошейте ESP.
- В Home Assistant установите HACS и интеграцию BLE Monitor (через HACS → Integrations).
- Добавьте интеграцию BLE Monitor: Configuration → Integrations → Add Integration → BLE Monitor → введите MAC весов → выберите параметры (weight, impedance, stabilized, battery).
- BLE Monitor создаст сенсоры в Home Assistant: `sensor.mi_scale_weight`, `sensor.mi_scale_impedance` и т.д.

5) Установка BodyMiScale (дополнительные расчёты)
- В HACS установите интеграцию `bodymiscale` или аналогичный компонент, если хотите автоматически получать BMI, fat%, muscle и т.п.
- Создайте файл `custom_components/bodymiscale/bodymiscale.yml` или следуйте инструкции компонента. Пример полей, которые потребуется заполнить: дата рождения, рост, пол, привязка сенсоров weight и impedance.

6) Подключение шаблонных сенсоров (расчёты / helpers)
- Скопируйте `config/sensors_body.yaml` и `config/helpers_body.yaml` в папку `/config/` Home Assistant.
- В `configuration.yaml` добавьте:
  ```yaml
  template: !include config/sensors_body.yaml
  input_number: !include config/helpers_body.yaml
  ```
- Перезагрузите Home Assistant и в UI заполните helpers (рост, возраст, пол).

7) Автоматизация (озвучивание)
- Откройте `automations/announce_mi_scale.yaml` или `automations/announce_mi_scale_debounce.yaml`.
- Замените `device_id` на `entity_id: media_player.YOUR_PLAYER_ENTITY_ID`.
- Рекомендуется использовать debounce-версию или добавить в триггер `for: 00:00:03` и условие, что разница веса больше 0.1 кг.
- Импортируйте автоматизацию или положите YAML в папку `automations/` и перезагрузите автоматизации.

8) Тестирование
- Поставьте весы на ровную поверхность, встаньте на них и дождитесь, когда показание стабилизируется (индикатор перестал мигать).
- В Home Assistant проверьте появление сенсоров: Developer Tools → States → найдите `sensor.mi_scale_weight` или `sensor.vesy_weight`.
- Запустите автоматизацию вручную: Developer Tools → Services → automation.trigger → выберите автоматизацию.

9) Отладка — что проверить при проблемах
- Весы не видны при сканировании: убедитесь, что вы пробудили весы и ESP32/сканер находится рядом.
- ESP не обнаруживает весы: выполните `esphome logs firmware/esp32_wroom_scale.yaml` и проверьте вывод BLE scan.
- Сенсоры не появились в HA: проверьте Integrations → BLE Monitor / Developer Tools → States.
- Автоматизация срабатывает часто: используйте debounce или условие delta > 0.1 кг.
- Некорректные числа: шаблоны в репо уже используют `replace(',', '.')` перед преобразованием в float.

10) Дополнительные заметки
- При первом запуске ESP создаёт Wi‑Fi точку, если он не может подключиться к вашей сети — подключитесь к ней и укажите Wi‑Fi данные.
- Стоимость набора (примерно): ~30 EUR (ESP32 + Mi Scale) — ориентировочно.
- Точность измерений: значения жира/мышц/воды — приблизительные; для медицинских целей используйте сертифицированные устройства.

11) Полезные команды
- Скан BLE (Linux):
  ```bash
  sudo hcitool lescan --duplicates
  ```
- Логи ESPHome:
  ```bash
  esphome logs firmware/esp32_wroom_scale.yaml
  ```
- Прошивка/сборка ESPHome:
  ```bash
  esphome run firmware/esp32_wroom_scale.yaml
  ```

---

Файлы в репозитории приведены к итоговому, практичному виду — в них нет вопросов от автора инструкции и нет предложений выполнить дополнительные действия. Этот документ содержит только пошаговые действия и проверки.