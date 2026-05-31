# Сценарии RuOpenRay

Этот репозиторий хранит обновляемые сценарии маршрутизации для RuOpenRay UI.

Сценарии больше не вшиваются в бинарник панели. При установке RuOpenRay
подключает этот репозиторий как обычный Git/raw-источник, а пользователь может
добавить свои источники в интерфейсе:

```text
Маршрутизация -> Сценарии -> Источники сценариев
```

Дефолтный URL каталога:

```text
https://raw.githubusercontent.com/AceAsket/RuOpenRay-scenarios/main/scenarios.json
```

## Что такое сценарий

Сценарий - это готовая подборка Xray routing rules с понятным названием,
описанием и иконкой. Например: Telegram, Discord, YouTube, Patreon,
Speedtest/Ookla или локальная RU-схема.

Идея простая:

- этот репозиторий можно обновлять без пересборки RuOpenRay UI;
- форки могут держать свои сценарии, названия и иконки;
- один каталог можно подключить к нескольким роутерам;
- пользовательские сценарии из UI не теряются при обновлении бинарника.

## Быстрый старт для своего каталога

1. Сделайте fork этого репозитория или создайте свой репозиторий.
2. Отредактируйте `scenarios.json`.
3. Проверьте, что файл доступен по raw-ссылке.
4. Добавьте raw URL в RuOpenRay:

```text
Маршрутизация -> Сценарии -> Источники сценариев -> Добавить источник
```

Для GitHub raw URL обычно выглядит так:

```text
https://raw.githubusercontent.com/USER/REPO/main/scenarios.json
```

## Формат каталога

Каталог - это обычный JSON:

```json
{
  "schema": 1,
  "name": "Мои сценарии RuOpenRay",
  "version": "2026-05-31",
  "scenarios": {
    "myService": {
      "title": "Мой сервис через proxy",
      "detail": "Короткое описание, которое будет видно в интерфейсе.",
      "icon": "mdi:rocket-launch",
      "rules": [
        {
          "type": "field",
          "outboundTag": "proxy",
          "domain": ["domain:example.com"]
        }
      ]
    }
  }
}
```

Поля верхнего уровня:

| Поле | Обязательно | Что означает |
| --- | --- | --- |
| `schema` | да | версия формата, сейчас `1` |
| `name` | да | название источника |
| `version` | желательно | версия или дата каталога |
| `description` | нет | описание источника |
| `scenarios` | да | объект со сценариями |

ID сценария, например `myService`, должен быть стабильным. Если переименовать
ID, RuOpenRay воспримет сценарий как новый.

## Поля сценария

| Поле | Обязательно | Что означает |
| --- | --- | --- |
| `title` | да | название карточки |
| `detail` | нет | описание под названием |
| `icon` | нет | Iconify-иконка или безопасный inline SVG |
| `rules` | да | массив Xray routing rules |

Пример сценария с несколькими правилами:

```json
{
  "title": "Telegram полный",
  "detail": "Домены Telegram, MTProto IP и UDP для звонков.",
  "icon": "simple-icons:telegram",
  "rules": [
    {
      "type": "field",
      "outboundTag": "proxy",
      "domain": ["domain:telegram.org", "domain:t.me", "regexp:.*\\.telegram\\.org"]
    },
    {
      "type": "field",
      "outboundTag": "proxy",
      "ip": ["91.108.4.0/22", "149.154.160.0/20"]
    },
    {
      "type": "field",
      "outboundTag": "proxy",
      "network": "udp",
      "ip": ["91.108.0.0/16", "149.154.160.0/20"]
    }
  ]
}
```

## Правила маршрутизации

Правила используют обычный Xray `field` routing shape. Поддерживаемые поля:

| Поле | Пример |
| --- | --- |
| `domain` | `["domain:example.com", "geosite:google", "regexp:.*\\.example\\.com"]` |
| `ip` | `["1.1.1.1", "8.8.8.0/24", "geoip:private"]` |
| `source` | `["192.168.1.20", "192.168.1.0/24"]` |
| `inboundTag` | `["tun-in", "socks-in"]` |
| `port` | `"443"` или `"80,443,8000-9000"` |
| `network` | `"tcp"`, `"udp"` или `"tcp,udp"` |
| `outboundTag` | `"proxy"`, `"direct"`, `"block"` или тег сервера |
| `balancerTag` | тег балансировщика |

Обычно сценарии лучше вести через `outboundTag: "proxy"`, потому что RuOpenRay
сам подставит текущий активный proxy-направление. Для прямого выхода используйте
`direct`, для блокировки - `block`.

## Иконки

В дефолтном каталоге для известных сценариев поле `icon` обычно не задано:
RuOpenRay сам подставляет красивую встроенную SVG-иконку по ID сценария.

Для своих сценариев самый простой вариант - Iconify name:

```json
"icon": "simple-icons:telegram"
```

Подходят, например:

- `simple-icons:discord`
- `simple-icons:youtube`
- `simple-icons:patreon`
- `simple-icons:speedtest`
- `circle-flags:ru`
- `mdi:shield-web`

Можно задать безопасный inline SVG:

```json
"icon": {
  "type": "svg",
  "background": "#ff6854",
  "foreground": "#ffffff",
  "svg": "<svg viewBox=\"0 0 24 24\"><path fill=\"currentColor\" d=\"...\"/></svg>"
}
```

RuOpenRay отклоняет SVG, если там есть `script`, event handlers, внешние
объекты, `javascript:` URL и другие небезопасные элементы. Цвет лучше задавать
через `currentColor`, а фон - через `background`.

## Приоритеты источников

Если один и тот же ID сценария есть в нескольких местах, RuOpenRay использует
такой порядок:

1. локальные пользовательские сценарии, созданные в UI;
2. подключенные Git/raw-источники сверху вниз;
3. встроенных сценариев нет.

Это позволяет форку переопределить сценарии из дефолтного источника, а
локальная правка в UI всё равно будет главнее.

## Ограничения и проверка

RuOpenRay проверяет каталог перед сохранением:

- максимум 200 сценариев в одном каталоге;
- максимум 1000 правил в одном каталоге;
- максимум 300 значений в одном списке правила;
- максимум 2 MiB на скачанный JSON;
- запрет небезопасного SVG;
- предупреждения по неизвестным `outboundTag`/`balancerTag`.

Схема JSON лежит в `scenarios.schema.json`, маленький пример - в
`examples/minimal.json`.

## CLI

На роутере источник можно добавить и обновить без браузера:

```sh
ruopenray-ui route-presets add-source \
  https://raw.githubusercontent.com/AceAsket/RuOpenRay-scenarios/main/scenarios.json \
  --name "RuOpenRay scenarios"

ruopenray-ui route-presets update
```

Автообновление источника можно включить при добавлении:

```sh
ruopenray-ui route-presets add-source URL --name "My scenarios" --auto-update
```

Установщик RuOpenRay использует переменные:

| Переменная | По умолчанию | Что делает |
| --- | --- | --- |
| `RUOPENRAY_INSTALL_SCENARIOS` | `1` | подключать дефолтный источник при установке |
| `RUOPENRAY_SCENARIOS_URL` | дефолтный raw URL этого репозитория | URL каталога |
| `RUOPENRAY_SCENARIOS_NAME` | `RuOpenRay scenarios` | название источника |
| `RUOPENRAY_SCENARIOS_AUTO_UPDATE` | `0` | включить ежедневное автообновление |

## Как добавлять сценарии в этот репозиторий

1. Добавьте сценарий в `scenarios.json`.
2. Используйте стабильный ID в camelCase, например `patreon` или `speedtestOokla`.
3. Держите `title` и `detail` на русском.
4. По возможности используйте узнаваемую иконку.
5. Не добавляйте слишком широкие правила без необходимости.
6. Проверьте JSON перед commit.

Хороший сценарий должен быть понятен человеку в интерфейсе: что он включает,
куда отправляет трафик и почему эти правила собраны вместе.
