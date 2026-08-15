# Home Assistant + Xiaomi Mi Scale 2

Подключаем Xiaomi Mi Scale 2 к Home Assistant через ESP32 и Bluetooth.

> **Важно:** расчёты состава тела по импедансу зависят от алгоритма весов и параметров пользователя. Репозиторий не выдаёт приближённые значения жира/мышц за медицинские измерения.

## Что получится

- ESP32 работает как Bluetooth Proxy для Home Assistant.
- Вес и импеданс можно получать в Home Assistant.
- ИМТ можно считать по весу и росту.
- Озвучивание результата через TTS — отдельная, необязательная часть.

## Быстрый старт

### 1. Подготовьте ESP32

Нужна ESP32 с BLE и установленный ESPHome. Откройте [`esphome/esp-bt.yaml`](esphome/esp-bt.yaml).

Создайте `secrets.yaml` и возьмите структуру из [`secrets.yaml.example`](secrets.yaml.example).

```yaml
wifi_ssid: "YOUR_WIFI_NAME"
wifi_password: "YOUR_WIFI_PASSWORD"
esphome_api_encryption_key: "YOUR_32_BYTE_BASE64_KEY"
esphome_ota_password: "YOUR_OTA_PASSWORD"
```

Прошейте устройство:

```bash
esphome run esphome/esp-bt.yaml
```

### 2. Подключите весы

Поставьте ESP32 рядом с весами и добавьте ESPHome-устройство в Home Assistant.

**Вариант A — `xiaomi_miscale2`**

Если ваша версия ESPHome поддерживает компонент `xiaomi_miscale2`, раскомментируйте его в [`esphome/esp-bt.yaml`](esphome/esp-bt.yaml) и укажите MAC-адрес весов.

**Вариант B — Bluetooth Proxy**

Оставьте `esp32_ble_tracker` + `bluetooth_proxy` и используйте BLE-интеграцию Home Assistant, которая умеет читать Mi Scale 2. Это хороший вариант, если нативный компонент недоступен.

### 3. Добавьте ИМТ (необязательно)

Скопируйте [`config/sensors_body.yaml`](config/sensors_body.yaml) в конфигурацию Home Assistant и подключите его так:

```yaml
template: !include config/sensors_body.yaml
```

Создайте helper `input_number.user_height` — рост в сантиметрах.

Конфиг ожидает сущность веса `sensor.vesy_weight`. Если у вас другое имя, измените его в файле.

### 4. Озвучивание (необязательно)

Автоматизации в [`home-assistant/automations/`](home-assistant/automations/) не нужны для базовой работы весов. Сначала убедитесь, что вес появляется в Home Assistant, затем подключайте TTS.

## Структура

```text
.
├── config/sensors_body.yaml
├── esphome/esp-bt.yaml
├── home-assistant/automations/
├── docs/
├── secrets.yaml.example
├── .gitignore
└── README.md
```

## Если не работает

1. Проверьте, что ESP32 подключён к Wi-Fi и виден в Home Assistant.
2. Уменьшите расстояние между ESP32 и весами.
3. Проверьте логи ESPHome и BLE-сканирование.
4. Проверьте точное имя сущности веса в Home Assistant.
5. Если используете `xiaomi_miscale2`, проверьте поддержку компонента в вашей версии ESPHome.
6. Не добавляйте настоящий `secrets.yaml` в Git.

Подробности: [`docs/troubleshooting.md`](docs/troubleshooting.md).

## Безопасность

Никогда не коммитьте Wi-Fi пароль, API encryption key, OTA пароль, токены или персональные данные. Для публичного репозитория используйте только `secrets.yaml.example`.

## Видео

Демонстрация проекта: https://youtube.com/shorts/LJl9GLgwjTk?feature=share

## Лицензия

MIT — [`LICENSE`](LICENSE).
