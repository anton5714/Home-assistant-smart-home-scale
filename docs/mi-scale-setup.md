# Xiaomi Mi Scale → Home Assistant

Короткая инструкция для подключения Xiaomi Mi Scale / Mi Scale 2 к Home Assistant через ESP32 и ESPHome.

## Требуется

- Xiaomi Mi Scale или Mi Scale 2;
- ESP32 с поддержкой Bluetooth;
- Home Assistant;
- ESPHome.

## 1. Подготовьте secrets

Скопируйте `secrets.yaml.example` в локальный `secrets.yaml` и заполните значения.

`secrets.yaml` не должен попадать в Git.

## 2. Узнайте MAC-адрес весов

Можно использовать nRF Connect или другой BLE scanner.

В `esphome/esp-bt.yaml` замените placeholder:

```yaml
mac_address: "AA:BB:CC:DD:EE:FF"
```

на MAC вашей модели.

## 3. Проверьте конфигурацию

```bash
esphome config esphome/esp-bt.yaml
```

После успешной проверки прошейте ESP32:

```bash
esphome run esphome/esp-bt.yaml
```

## 4. Проверьте Home Assistant

После добавления ESPHome-устройства должны появиться:

- `sensor.mi_scale_weight` — вес;
- `sensor.mi_scale_impedance` — импеданс, если его передаёт модель.

Для Mi Scale 2 используется компонент ESPHome `xiaomi_miscale`. Не используйте старое имя `xiaomi_miscale2`.

## 5. Если данные не приходят

Проверьте по порядку:

1. ESP32 находится достаточно близко к весам.
2. MAC-адрес указан правильно.
3. Весы действительно передают BLE-рекламу во время измерения.
4. В логах ESPHome нет ошибок BLE/API.
5. В Home Assistant устройство ESPHome находится online.

Не добавляйте сторонние BLE-компоненты, пока не проверили штатный `xiaomi_miscale`.
