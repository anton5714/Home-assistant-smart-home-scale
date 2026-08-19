# Xiaomi Mi Scale → Home Assistant

Connect a Xiaomi Mi Scale or Mi Scale 2 to Home Assistant using an ESP32 and Bluetooth Low Energy.

[![ESPHome](https://img.shields.io/badge/ESPHome-supported-blue)](https://esphome.io/)
[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-supported-blue)](https://www.home-assistant.io/)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)

## 🎥 See it in action

Want to see how the setup works before installing it?

**Watch the short demo:**

[▶️ Xiaomi Mi Scale → Home Assistant — demo on YouTube](https://www.youtube.com/shorts/LJl9GLgwjTk)

The video shows the real setup and how the scale sends measurements to Home Assistant through ESP32 + Bluetooth.

## What you get

- Weight sensor in kg
- Impedance sensor on Mi Scale 2
- Passive BLE reception — no pairing with the scale
- ESPHome API connection to Home Assistant
- Optional ESP32 Bluetooth Proxy
- Optional BMI sensor

ESPHome's `xiaomi_miscale` component listens to BLE advertisement packets passively, so the scale does not need to be paired. Mi Scale 2 can provide both weight and impedance.

## Architecture

```text
Xiaomi Mi Scale
      │
      │ Bluetooth LE
      ▼
    ESP32
      │
      │ ESPHome API
      ▼
Home Assistant
      │
      ├── Weight
      ├── Impedance
      └── Optional BMI
```

## Requirements

- Xiaomi Mi Scale or Mi Scale 2
- ESP32 board with BLE support
- Home Assistant
- ESPHome

## Quick start

### 1. Prepare secrets

Copy `secrets.yaml.example` to your local ESPHome secrets file and fill in your Wi-Fi/API/OTA values.

**Never commit the real `secrets.yaml`.**

### 2. Set the scale MAC address

Edit `esphome/esp-bt.yaml`:

```yaml
mac_address: "AA:BB:CC:DD:EE:FF"
```

Replace the placeholder with your scale's Bluetooth MAC address.

### 3. Flash the ESP32

```bash
esphome run esphome/esp-bt.yaml
```

### 4. Add ESPHome to Home Assistant

After the device connects, Home Assistant should expose:

- `Mi Scale Weight`
- `Mi Scale Impedance` — Mi Scale 2 only
- ESP uptime
- Wi-Fi signal
- Restart button

The exact entity IDs can differ depending on the device name and Home Assistant configuration.

## Mi Scale vs Mi Scale 2

| Feature | Mi Scale | Mi Scale 2 |
|---|---:|---:|
| Weight | ✅ | ✅ |
| Impedance | — | ✅ |
| BLE passive reception | ✅ | ✅ |
| Pairing required | ❌ | ❌ |

## Optional BMI

The repository includes an optional Home Assistant template sensor in `home-assistant/body_metrics.yaml`.

Create an `input_number.user_height_cm` helper and include the template in your Home Assistant configuration.

BMI is calculated from weight and height only. The project intentionally does **not** invent body-fat, muscle-mass or bone-mass values using arbitrary coefficients.

## Troubleshooting

### Weight does not appear

1. Confirm the MAC address.
2. Confirm the ESP32 is close enough to the scale.
3. Check ESPHome logs.
4. Step on the scale and wait for a stable measurement.
5. Make sure `esp32_ble_tracker` is enabled.

### Impedance is missing

Impedance is supported by Mi Scale 2, not the original Mi Scale.

### ESP32 connects but Home Assistant has no entities

Check the ESPHome API connection and inspect the device logs. The repository uses encrypted ESPHome API credentials from `secrets.yaml`.

## Development

The repository includes GitHub Actions that build the ESPHome configuration on pushes and pull requests.

You can also validate locally:

```bash
esphome config esphome/esp-bt.yaml
```

## Project structure

```text
.
├── .github/
│   └── workflows/
│       └── esphome.yml
├── esphome/
│   └── esp-bt.yaml
├── home-assistant/
│   └── body_metrics.yaml
├── docs/
│   ├── firmware-secrets.md
│   ├── mi-scale-setup.md
│   └── sensors-setup.md
├── secrets.yaml.example
├── CONTRIBUTING.md
├── SECURITY.md
├── LICENSE
└── README.md
```

## Security

Do not commit passwords, API keys, tokens or private credentials. See `SECURITY.md` for the security policy.

## License

MIT
