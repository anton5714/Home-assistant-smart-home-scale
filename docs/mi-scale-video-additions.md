# Дополнения к инструкции по видео (YouTube: How to Make Xiaomi Mi Scale 2 Smart with ESP32 and Home Assistant)

Это дополнение содержит ключевые моменты из видео и важные шаги/проверки, которые часто пропускают — я добавил их, чтобы инструкция в репозитории стала эталонной и максимально понятной.

1) Быстрое резюме из видео — что важно
- Пробуждайте весы (встаньте или коснитесь), прежде чем сканировать BLE — иначе устройство не появится.  
- Нужно дождаться, когда показание стабилизируется (мигающие индикаторы закончились) — именно тогда значение корректно.  
- Получив MAC устройства, можно прописать его в конфиг ESPHome или в BLE Monitor/интеграцию.  
- Рекомендуемый путь: ESP32 с `esp32_ble_tracker` + `bluetooth_proxy` → HACS: BLE Monitor (или bodymiscale) в Home Assistant для парсинга пакетов и создания sensora.

2) Как найти MAC адрес весов (пошагово)
- Вариант A (мобильное приложение/сканер): установите nRF Connect (Android/iOS) → включите сканирование → пробудите весы → ищите устройство с именем похожим на `MIBCS`/`Xiaomi` и запишите MAC.
- Вариант B (Linux): подойдёт устройство с Bluetooth:
  sudo btmgmt find
  или
  sudo hcitool lescan --duplicates
  Затем пробудите весы и найдите строку с MAC.

3) ESPHome — пример корректной конфигурации для Mi Scale 2 (если поддерживается)
- Начиная с последних версий ESPHome в нативном функционале может быть поддержка Mi Scale 2; пример:

```yaml
esphome:
  name: mi-scale-bridge

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
    mac_address: 'AA:BB:CC:DD:EE:FF' # замените на MAC ваших весов
    name: "Mi Scale 2 Weight"
    update_interval: 10s
    weight:
      name: "Mi Scale Weight"
    impedance:
      name: "Mi Scale Impedance"
```

- Если `xiaomi_miscale2` не доступен в вашей версии ESPHome — используйте `esp32_ble_tracker` + `bluetooth_proxy` и парсите пакеты на стороне Home Assistant через BLE Monitor / custom component.

4) HACS + BLE Monitor / bodymiscale (как установить и настроить)
- Установите HACS (если ещё не установлен): https://hacs.xyz/docs/installation/manual
- В HACS → Integrations → поищите "BLE Monitor" и/или "bodymiscale" и установите.  
- После установки перезагрузите Home Assistant и добавьте интеграцию через Configuration → Integrations → Add Integration → найдите BLE Monitor / BodyMiScale и следуйте мастеру.  
- Для BLE Monitor добавьте MAC весов и выберите какие параметры парсить (weight, impedance, battery).

5) Настройка автоматического распознавания пользователя (user detection)
- В репозитории есть шаблонные sensors. Для автоприсвоения можно использовать один из подходов:
  - AppDaemon/пользовательская логика: создайте профиль для каждого человека с min/max весом и сопоставляйте показания.  
  - Template sensors в Home Assistant: заведите отдельные template сущности с условием (например, если weight > X and weight < Y → user = Anton).  
- Внимание: диапазоны не должны сильно пересекаться; при схожих весах используйте дополнительную логику (время измерения, предыдущая запись, Bluetooth presence).  

6) Стабилизация показания — почему важно
- Весы передают «сырые» данные во время взвешивания, пока пользователь не успокоился; если отправить в HA слишком рано, получится плавающее/неверное значение.  
- Рекомендуется в автоматизации использовать задержку `for:` в триггере (например, 3 сек) или проверять, что в пакете стоит флаг «stabilized». BLE Monitor часто предоставляет информацию о стабилизации.

7) MQTT / discovery / firewall — распространённые проблемы
- Если вы используете MQTT, убедитесь, что broker доступен и включена discovery (если хотите автоподключение).  
- Проверьте настройки firewall/маршрутизатора: порты MQTT (1883) должны быть открыты внутри сети и для устройств, если используете отдельный брокер.  

8) Компиляция/память ESP32 — возможная проблема
- Если при компиляции ESPHome получаете ошибки «not enough space» или ошибки линковки, попробуйте сменить board на `esp32doit-devkit-v1` или увеличить partition, или использовать минимальную spiffs таблицу.  
- Пример настройки partition: в `platformio` или в YAML — указывать `board_build.partitions` (требует advanced setup). Если не уверены — сообщите модель платки, я помогу.

9) Troubleshooting — чеклист (что проверить при проблемах)
- Весы не видны в сканере: пробудите весы физически (встаньте/нажмите).  
- ESP не получает данные: проверьте логи `esphome logs <config>` и убедитесь, что `esp32_ble_tracker` обнаруживает MAC.  
- В HA нет сенсоров: проверьте интеграцию BLE Monitor / bodymiscale; также посмотрите `Developer Tools → States`.  
- Автоматизация срабатывает слишком часто: используйте debounce (announce_mi_scale_debounce.yaml) или условия сравнения (delta > 0.1кг).  
- Неверные единицы/запятые: в шаблонах используйте replace(',', '.') перед приведением к float.

10) Полезные команды и утилиты
- Скан BLE (Linux): `sudo hcitool lescan --duplicates`  
- Логи ESPHome: `esphome logs firmware/esp32_wroom_scale.yaml`  
- Сборка/прошивка: `esphome run firmware/esp32_wroom_scale.yaml`  

11) Рекомендации по документации в репо (что я добавил)
- Инструкция по поиску MAC (nRF Connect / hcitool) — шаги добавлены.  
- Пример ESPHome-конфига с platform: xiaomi_miscale2 (если поддерживается).  
- Рекомендации по HACS → BLE Monitor / bodymiscale и как их настроить.  
- Troubleshooting чеклист и советы по стабилизации показаний.  

12) Следующие шаги — могу сделать за вас
- Включить пример кастомного ESPHome-парсера, если нужен (C++ snippet) — скажите «парсер».  
- Обновить automations, чтобы они явно проверяли stabilized или использовали сенсор stabilized из BLE Monitor.  
- Подготовить UI/Lovelace-дашборд с карточками веса/графиками/историей.

Если хотите, я сейчас закомичу эти дополнения в репозиторий (файл docs/mi-scale-video-additions.md) — подтвердите, и я сделаю коммит.