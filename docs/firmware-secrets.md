Файл: firmware/esp32_wroom_scale.yaml

Этот файл — пример конфигурации ESPHome для ESP32 (Wroom) с включённым BLE-сканером.
Я заменил все секреты на ссылки в `secrets.yaml`, чтобы файл можно было публично хранить в репозитории.

Какие секреты нужно добавить в вашу secrets.yaml (Home Assistant):

```yaml
wifi_ssid: "ваша_wifi_сеть"
wifi_password: "ваш_wifi_пароль"
esphome_api_encryption_key: "случайная_строка_для_API_шифрования"
esphome_ota_password: "случайная_пароль_для_OTA"
```

Как сгенерировать безопасные значения:
- Для `esphome_api_encryption_key` используйте 32+ байтовую строку, закодированную base64 или случайные символы. Например (Linux/macOS):
  `head -c 32 /dev/urandom | base64`
- Для `esphome_ota_password` используйте длинную случайную строку:
  `openssl rand -base64 18`

Где положить secrets.yaml:
- Если вы используете Home Assistant, добавьте эти строки в файл `secrets.yaml` в каталоге конфигурации (обычно `/config/secrets.yaml`).
- Не добавляйте `secrets.yaml` в репозиторий — он должен быть в .gitignore (у нас уже есть .gitignore).

Как прошить устройство (быстро):
1) Через ESPHome (рекомендуется):
   - Установите ESPHome (доступно как аддон в Home Assistant или через pip).
   - Копируйте `firmware/esp32_wroom_scale.yaml` в ваш локальный каталог esp-home или используйте редактор ESPHome в Home Assistant.
   - Запустите: `esphome run firmware/esp32_wroom_scale.yaml` и следуйте инструкциям для прошивки по USB или OTA.

2) Через ESPHome Flasher (USB):
   - Подключите ESP32 к компьютеру по USB.
   - Скомпилируйте бинарник через `esphome compile firmware/esp32_wroom_scale.yaml`.
   - Загрузите бинарник в устройство через ESPHome Flasher или esptool.
