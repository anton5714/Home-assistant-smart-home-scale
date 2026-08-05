# Подключение Xiaomi Mi Scale 2 — кратко

Это сокращённая одностраничная инструкция. Полная версия находится в README.md.

Требуется: ESP32, Mi Scale 2, Home Assistant, ESPHome.

Шаги:
- Подготовьте secrets.yaml на основе secrets.yaml.example
- Прошейте esphome/esp-bt.yaml
- Найдите MAC весов (nRF Connect или hcitool)
- Включите xiaomi_miscale2 (если доступен) или используйте bluetooth_proxy + BLE Monitor
- Добавьте шаблонные сенсоры и автоматизацию
- Тестируйте: встаньте на весы, дождитесь стабилизации, проверьте сенсор
