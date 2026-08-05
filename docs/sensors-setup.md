# Как подключить template sensors для анализа тела

Файл: `config/sensors_body.yaml`

Шаги для включения в Home Assistant:

1) Скопируйте `config/sensors_body.yaml` в папку `config/` вашего репозитория (или в директорию конфигурации Home Assistant).

2) В `configuration.yaml` добавьте строку (если у вас Home Assistant Core / OS с поддержкой include):

```yaml
template: !include config/sensors_body.yaml
```

Если у вас уже есть блок `template:` в `configuration.yaml`, объедините определения или импортируйте отдельный файл с помощью `!include`.

3) Перезагрузите Home Assistant или выполните "Reload Template Entities" через Configuration → Server Controls → Reload Template Entities.

Проверка:
- После перезагрузки появятся сенсоры с указанными `unique_id` и именами.
- Проверьте значения в Developer Tools → States и в логах.

Замечания и рекомендации:
- Некоторые формулы (например, BMI) используют константы. Если у вас есть рост в метрах, замените `weight / 3.24` на `weight / (height * height)` и передавайте `height` как input_number или sensor.
- Убедитесь, что `sensor.vesy_weight` и `sensor.vesy_impedance` реально существуют и обновляются. В шаблонах уже используется `replace(',', '.')` для безопасного преобразования.
- По возможности используйте `unit_of_measurement` (как указано) — это улучшит отображение в интерфейсе.
- Если нужно — могу добавить автоматическое картографирование названий entity (alias / mapping) и проверку наличия сенсоров.

Безопасность:
- Не храните секреты/ключи в репозитории. Этот файл не содержит секретов.

Если хотите — могу создать UI-friendly version (input_number для роста) и обновить формулы (BMI, BMR с учётом возраста/роста/пола).