# Xiaomi Mi Scale → Home Assistant

Подключение **Xiaomi Mi Scale / Mi Scale 2** к Home Assistant через ESP32 и Bluetooth Low Energy.

Вот так это работает:
https://youtube.com/shorts/LJl9GLgwjTk?si=N0dysm-EmuSxpZ7c

## Что работает

- ⚖️ вес в кг;
- 🧬 импеданс в Ω на Mi Scale 2;
- 📡 BLE через ESP32;
- 🏠 Home Assistant API;
- 🔄 Bluetooth proxy;
- 📊 опциональный BMI в Home Assistant.

ESPHome принимает данные от весов пассивно через BLE; сопряжение с весами не требуется. Mi Scale 2 поддерживает передачу веса и импеданса.

## Быстрый старт

### 1. ESP32

Используйте `esphome/esp-bt.yaml`.

В `secrets.yaml` добавьте значения из `secrets.yaml.example`.

### 2. MAC-адрес весов

В `esphome/esp-bt.yaml` замените:

```yaml
mac_address: "AA:BB:CC:DD:EE:FF"
```

на MAC вашей Mi Scale.

После этого прошейте ESP32 через ESPHome:

```bash
esphome run esphome/esp-bt.yaml
```

### 3. Home Assistant

После подключения ESP32 интеграция ESPHome создаст сенсоры:

- `sensor.mi_scale_weight`
- `sensor.mi_scale_impedance` — для Mi Scale 2

### 4. BMI — опционально

Если нужен BMI, создайте в Home Assistant helper `input_number.user_height_cm` и подключите:

```yaml
template: !include home-assistant/body_metrics.yaml
```

BMI рассчитывается только из веса и роста. Проценты жира, мышечная и костная масса намеренно не подставляются приблизительными формулами: такие значения лучше брать непосредственно из данных весов или специализированной интеграции.

## Структура

```text
.
├── esphome/
│   └── esp-bt.yaml              # основной ESP32 + Mi Scale BLE конфиг
├── home-assistant/
│   └── body_metrics.yaml        # опциональный BMI
├── docs/
│   ├── mi-scale-setup.md
│   ├── firmware-secrets.md
│   └── sensors-setup.md
├── secrets.yaml.example
├── CONTRIBUTING.md
├── LICENSE
└── README.md
```

## Безопасность

Никогда не коммитьте настоящий `secrets.yaml`, Wi-Fi пароль или OTA/API ключи.

Не используйте статический IP, если он не нужен: конфигурация ESP32 должна работать в разных сетях без изменения YAML.

## Примечание про Mi Scale 2

В актуальном ESPHome компонент называется `xiaomi_miscale`; старый пример с `xiaomi_miscale2` больше не нужен.

## Источник

Документация ESPHome: https://esphome.io/components/sensor/xiaomi_miscale/

## License

MIT
