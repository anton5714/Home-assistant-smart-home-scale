# Подключение Xiaomi Mi Scale 2 к Home Assistant через ESP32 (Bluetooth)

Краткая, ясная и только пошаговая инструкция. Никаких комментариев от нейросети — только практические шаги.

Требуется
- ESP32 (например, WROOM/DevKitC)
- Xiaomi Mi Scale 2
- Home Assistant (работающая инстанция)
- Компьютер для прошивки ESP32 или ESPHome аддон в Home Assistant
- Доступ к файлу `secrets.yaml` (локально, не пушьте в репозиторий)

1) Клонирование (опционально)
- Если вы работаете с этим репозиторием локально:
  git clone https://github.com/anton5714/Home-assistant-smart-home-scale.git
  cd Home-assistant-smart-home-scale

2) Подготовьте secrets.yaml (локально, не в репо)
- В конфигурации Home Assistant (`/config/secrets.yaml`) добавьте:
  wifi_ssid: "ВАША_WIFI"
  wifi_password: "ВАШ_WIFI_PASSWORD"
  esphome_api_encryption_key: "случайная_строка"
  esphome_ota_password: "случайная_строка"

3) Получение MAC-адреса весов
- Пробудите весы (коснитесь или встаньте на них).
- Вариант A (мобильный): используйте nRF Connect (Android/iOS) → Start scan → найдите устройство с именем типа `MIBCS`/`Mi` → запишите MAC.
- Вариант B (Linux): выполните в терминале:
  sudo hcitool lescan --duplicates
  затем пробудите весы и запишите появившийся MAC (формат AA:BB:CC:DD:EE:FF).

4) Вариант прошивки и парсинга
4a) Вариант (если ESPHome поддерживает xiaomi_miscale2)
- Отредактируйте `firmware/esp32_wroom_scale.yaml` или создайте конфиг и добавьте:

```yaml
esphome:
  name: mi-scale-esp

esp32:
  board: esp32dev

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

esp32_ble_tracker:

bluetooth_proxy:
  active: true

sensor:
  - platform: xiaomi_miscale2
    mac_address: 'AA:BB:CC:DD:EE:FF' # ваш MAC
    name: "Mi Scale 2"
    update_interval: 10s
    weight:
      name: "Mi Scale Weight"
    impedance:
      name: "Mi Scale Impedance"
```

- Прошейте устройство: `esphome run firmware/esp32_wroom_scale.yaml` (или через ESPHome аддон).

4b) Вариант (общий, если xiaomi_miscale2 недоступен)
- Используйте esp32_ble_tracker + bluetooth_proxy, а парсинг делайте на стороне Home Assistant через BLE Monitor (HACS) или bodymiscale:

```yaml
esp32_ble_tracker:

bluetooth_proxy:
  active: true
```

- Прошейте ESP32 с этим config. В Home Assistant установите через HACS интеграцию BLE Monitor и добавьте MAC весов — BLE Monitor создаст `sensor.*` для веса и импеданса.

5) Установка HACS и BLE Monitor (если выбираете вариант 4b)
- Установите HACS: следуйте официальной инструкции https://hacs.xyz/docs/installation/manual
- В HACS → Integrations → найдите и установите "BLE Monitor" (и при желании "bodymiscale").
- Перезагрузите Home Assistant.
- Добавьте интеграцию BLE Monitor: Configuration → Integrations → Add Integration → BLE Monitor → укажите MAC весов → выберите парсинг weight/impedance/stabilized.

6) Подключение шаблонных сенсоров (опционально — расчёты)
- Скопируйте `config/sensors_body.yaml` и `config/helpers_body.yaml` в `/config/` Home Assistant (если хотите вычислять BMI, fat% и т.д.).
- В `configuration.yaml` добавьте:
  template: !include config/sensors_body.yaml
  input_number: !include config/helpers_body.yaml
- Перезагрузите Home Assistant.

7) Добавление автоматизации (озвучивание)
- Откройте `automations/announce_mi_scale.yaml` или `automations/announce_mi_scale_debounce.yaml`.
- Замените `device_id` на `entity_id: media_player.имя_плеера` или укажите ваш media_player.
- Импортируйте автоматизацию через UI или помес��ите YAML в папку `automations/` и Reload Automations.
- Рекомендуется использовать debounce версию или добавить условие изменения > 0.1 кг и `for: 00:00:03`.

8) Тестирование
- Поставьте весы на ровную поверхность, встаньте на них и дождитесь стабилизации показаний (индикатор перестал мигать).
- В Home Assistant проверьте появление сущностей: Developer Tools → States → найдите `sensor.mi_scale_weight` или `sensor.vesy_weight`.
- Вручную запустите автоматизацию (Developer Tools → Services → automation.trigger) и проверьте воспроизведение на `media_player`.

9) Быстрая отладка (если что-то не работает)
- Весы не видны в сканере: убедитесь, что вы пробудили весы и что ESP32/мобильный сканер находится поблизости.
- ESPHome не видит весы: запустите `esphome logs firmware/esp32_wroom_scale.yaml` и смотрите обнаружение MAC.
- Сенсоры не появились в HA: проверьте Integrations → BLE Monitor / Developer Tools → States.
- Автоматизация срабатывает часто: используйте debounce (announce_mi_scale_debounce.yaml) или условие delta > 0.1 кг.
- Неправильные числа (запятые): шаблоны в репо уже используют `replace(',', '.')` перед преобразованием в float.

10) Полезные команды
- Скан BLE (Linux): `sudo hcitool lescan --duplicates`
- Логи ESPHome: `esphome logs firmware/esp32_wroom_scale.yaml`
- Сборка/прошивка: `esphome run firmware/esp32_wroom_scale.yaml`

11) Примечания по безопасности
- Никогда не публикуйте `secrets.yaml` в репозиторий.
- Если вы тестировали с реальными ключами в публичных файлах — смените их (ротация).

---

Если нужно, могу: 1) заменить/обновить автоматизацию чтобы явно проверять stabilized/debounce; 2) добавить пример Lovelace‑карты для отображения веса и графиков; 3) сгенерировать QR‑код для крипто‑адреса и добавить в repo. Скажите, какой из пунктов выполнить.