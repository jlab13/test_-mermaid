# Базова схема програмного модуля

## Перелік прийнятих умовних позначень та скорочень

* БМ - Бойовий мдуль (python daemon)
* ПК - Пульт керування
* МО - Монітор оператора
* РШ - Регулятор швидкості ESC
* ТК - Тонкий кліент APP
* ТУ - Турель (енкодери положення)

## Взаємодія модулів по шині CAN - DSDL

```mermaid

flowchart LR
    БМ("БМ - Бойовий мдуль (python daemon)")
    ПК(ПК - Пульт керування)
    МО(МО - Монітор оператора)
    РШ(РШ - Регулятор швидкості)
    ТК(ТК - Тонкий кліент APP)
    ТЛ(ТУ - Турель)

    ПК --> | orion.input.Event | БМ
    МО --> | orion.input.Event | БМ
    БМ --> | orion.output.Led | ПК
    БМ --> | orion.output.Led | МО
    БМ --> | reg.udral.service.actuator.common.sp.Vector2 | РШ
    БМ --> | orion.system.Mode | ТК
    ТЛ --> | uavcan.si.unit.angle.Scalar | ТК
    ТЛ --> | uavcan.si.unit.angle.Scalar | БМ
```

## Запуск системи

```mermaid
sequenceDiagram
    participant БМ
    participant МКК
    participant ТК

    loop Старт системи
        МКК->>БМ: uavcan.node.Heartbeat
        ТК->>БМ: uavcan.node.Heartbeat
    end
    loop Режим роботи системи
        БМ->>МКК: orion.system.Mode
        БМ->>ТК: orion.system.Mode
    end
```

## БМ - Бойовий мдуль / *State machine*

```mermaid

stateDiagram-v2
    init: Ініціалізація
    view: Спостереження
    arms: Бойовий
    load: Заряджання

    [*] --> init
    init --> view
    state view {
        view_fn: Фіункції 

        [*] --> view_fn
        [*] --> ОЕМ
        state view_fn {
            contrast: Контраст ТПК
            brightness: Яскравість ТПК
            focus: Фокус
            color: Схема кольорів

            [*] --> contrast
            contrast --> brightness
            brightness --> focus
            focus --> color
            color --> contrast
        }
        state ОЕМ {
            [*] --> ТВК
            ТВК --> x1
            x1 --> x2
            x2 --> x4
            x4 --> x1
            ТВК --> ТПК
            ТПК --> ТВК
        }
    }

    init --> arms
    init --> load
    state load {
        load_fn: Фіункції
        add: БК+
        del: БК-
        
        [*] --> load_fn
        state load_fn {
            [*] --> add
            add --> del
            del --> add
        }

    }

```
