# Умный дом: Xiaomi Mi Scale 2 → ESP32 → Алиса

ESP32 читает данные с Xiaomi Mi Scale 2 по Bluetooth и отправляет их в Home Assistant, колонка с Алисой озвучивает показатели и даёт краткие рекомендации.

В этом репозитории вы найдёте:
- automations/ — YAML автоматизаций для Home Assistant (включая debounce‑версию)
- docs/ — пошаговые инструкции по подключению и настройке
- firmware/ — примерный esphome конфиг (с placeholders)
- images/ — (пока пусто) фотографии и схемы
- DONATE.md — как поддержать проект

Быстрый старт
1. Клонируйте репозиторий:
   git clone https://github.com/anton5714/Home-assistant-smart-home-scale.git
2. Проверьте папку automations/ и замените device_id/entity_id и имена сенсоров на свои.
3. Импортируйте YAML в Home Assistant (через UI или положите файлы в папку automations/ конфигурации).
4. Прошейте ESP32 (см. firmware/esphome_scale.yaml) и убедитесь, что в HA появились сенсоры.

Поддержать проект
См. DONATE.md

Безопасность
- Никогда не храните в репозитории токены и ключи. Используйте secrets.yaml или переменные окружения.

Если хотите, могу автоматически создать страницу GitHub Pages из содержимого docs/ и добавить бейджи донатов.
