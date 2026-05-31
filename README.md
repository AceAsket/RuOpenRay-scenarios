# RuOpenRay scenarios

This repository contains updateable routing scenarios for RuOpenRay UI.
The UI binary does not contain these scenarios. The OpenWrt installer imports
`scenarios.json` as a regular Git/raw source, and users can add their own
catalog URLs from the web interface.

По-русски коротко: сценарий - это JSON-каталог с красивым названием,
иконкой и обычными Xray routing rules. Чтобы добавить свои подборки, форкните
репозиторий или положите такой же `scenarios.json` в любой Git/raw URL, затем
добавьте ссылку в RuOpenRay: Маршрутизация -> Сценарии -> Источники сценариев.

Default catalog URL:

`https://raw.githubusercontent.com/AceAsket/RuOpenRay-scenarios/main/scenarios.json`

## Catalog format

A catalog is plain JSON:

```json
{
  "schema": 1,
  "name": "My RuOpenRay scenarios",
  "version": "2026-05-31",
  "scenarios": {
    "myService": {
      "title": "My service through proxy",
      "detail": "Human-readable description.",
      "icon": "mdi:rocket-launch",
      "rules": [
        { "type": "field", "outboundTag": "proxy", "domain": ["domain:example.com"] }
      ]
    }
  }
}
```

Scenario IDs must be stable. If two connected sources define the same ID,
RuOpenRay uses this priority: local UI scenarios, Git/raw sources from top to
bottom, then nothing else. The binary has no default scenario catalog.

## Icons

Use either an Iconify name:

```json
"icon": "simple-icons:telegram"
```

or safe inline SVG:

```json
"icon": {
  "type": "svg",
  "background": "#ff6854",
  "foreground": "#ffffff",
  "svg": "<svg viewBox=\"0 0 24 24\"><path fill=\"currentColor\" d=\"...\"/></svg>"
}
```

RuOpenRay rejects SVG with scripts, event handlers, external objects and
`javascript:` URLs.

## Rules

Rules use Xray `field` routing shape. Supported fields:

- `domain`: array of Xray domain matchers, for example `domain:example.com`, `geosite:google`, `regexp:.*\.example\.com`.
- `ip`: array of IP/CIDR/geoip matchers.
- `source`: LAN client IP/CIDR matchers.
- `inboundTag`: inbound tags.
- `port`: Xray port expression.
- `network`: `tcp`, `udp`, or `tcp,udp`.
- `outboundTag`: target outbound, usually `proxy`, `direct`, or `block`.
- `balancerTag`: target balancer instead of outbound.

Limits enforced by RuOpenRay: 200 scenarios per catalog, 1000 total rules,
300 values per rule list, and 2 MiB max catalog download size.

See `examples/minimal.json` for a small custom catalog.
