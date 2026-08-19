# BMI в Home Assistant

Проект намеренно считает только BMI из веса и роста. Не используйте самодельные коэффициенты для жира, воды или мышечной массы: такие оценки зависят от модели весов и алгоритма производителя.

## Подключение

Файл: `home-assistant/body_metrics.yaml`

Добавьте в `configuration.yaml`:

```yaml
template: !include home-assistant/body_metrics.yaml
```

Создайте Helper `input_number.user_height_cm` с ростом в сантиметрах.

После изменения конфигурации проверьте её и перезагрузите Template entities в Home Assistant.

## Что появится

- `sensor.mi_scale_bmi` — BMI;
- `sensor.mi_scale_weight_change` — изменение относительно заданного reference weight.

## Важно

Если ваши entity IDs отличаются от примеров, измените их в `home-assistant/body_metrics.yaml`.
