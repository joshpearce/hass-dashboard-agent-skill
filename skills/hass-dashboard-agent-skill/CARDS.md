# Card Types Reference

Complete reference for Home Assistant Lovelace card types.

## Tile Card

**Recommended for most entities.** Compact display with optional controls.

```json
{
  "type": "tile",
  "entity": "light.living_room",
  "name": "Living Room Light",
  "icon": "mdi:lamp",
  "color": "amber",
  "vertical": false,
  "hide_state": false,
  "state_content": ["state", "brightness"],
  "features": [
    { "type": "light-brightness" }
  ],
  "tap_action": { "action": "toggle" },
  "hold_action": { "action": "more-info" }
}
```

### Tile Options

| Option | Type | Description |
|--------|------|-------------|
| `entity` | string | Entity ID (required) |
| `name` | string | Override display name |
| `icon` | string | Override icon |
| `color` | string | Color when active (token or hex) |
| `vertical` | boolean | Stack icon above name (more compact for dense layouts) |
| `hide_state` | boolean | Hide state text |
| `show_entity_picture` | boolean | Use entity picture |
| `state_content` | string/array | What to show: `state`, `last_changed`, `last_updated`, or attribute |
| `features` | array | Control widgets (see below) |
| `features_position` | string | `bottom` or `inline` |
| `tap_action` | object | Action on tap |
| `hold_action` | object | Action on hold |
| `grid_options` | object | Grid sizing: `{ "columns": 6, "rows": 2 }` |

### Tile Features

| Feature Type | Use For |
|--------------|---------|
| `light-brightness` | Brightness slider for lights |
| `light-color-temp` | Color temperature control |
| `fan-speed` | Fan speed control |
| `cover-open-close` | Cover open/close buttons |
| `cover-position` | Cover position slider |
| `climate-hvac-modes` | HVAC mode selector |
| `climate-preset-modes` | Climate preset selector |

### Available Colors

`primary`, `accent`, `disabled`, `red`, `pink`, `purple`, `deep-purple`, `indigo`, `blue`, `light-blue`, `cyan`, `teal`, `green`, `light-green`, `lime`, `yellow`, `amber`, `orange`, `deep-orange`, `brown`, `grey`, `blue-grey`, `black`, `white`, or hex codes (e.g., `#93c47d`)

---

## Entities Card

List of entities with optional header and toggle.

```json
{
  "type": "entities",
  "title": "Living Room",
  "icon": "mdi:sofa",
  "show_header_toggle": true,
  "state_color": true,
  "entities": [
    "light.living_room",
    {
      "entity": "switch.tv",
      "name": "Television",
      "icon": "mdi:television",
      "secondary_info": "last-changed"
    }
  ]
}
```

### Entity Row Options

| Option | Type | Description |
|--------|------|-------------|
| `entity` | string | Entity ID (required) |
| `name` | string | Override name |
| `icon` | string | Override icon |
| `secondary_info` | string | `entity-id`, `last-changed`, `last-updated`, `brightness` |
| `format` | string | For timestamps: `relative`, `total`, `date`, `time`, `datetime` |
| `state_color` | boolean | Color icon by state |

### Special Rows

```json
{
  "entities": [
    { "type": "divider" },
    { "type": "section", "label": "Section Header" },
    {
      "type": "button",
      "name": "Run Script",
      "action_name": "Execute",
      "tap_action": {
        "action": "call-service",
        "service": "script.my_script"
      }
    },
    {
      "type": "weblink",
      "name": "Home Assistant",
      "url": "https://www.home-assistant.io"
    }
  ]
}
```

---

## Heading Card

Section headers within a view.

```json
{
  "type": "heading",
  "heading": "Section Title",
  "heading_style": "title"
}
```

| Option | Values |
|--------|--------|
| `heading_style` | `title`, `subtitle` |

---

## Gauge Card

Visual gauge for numeric sensors.

```json
{
  "type": "gauge",
  "entity": "sensor.cpu_usage",
  "name": "CPU Usage",
  "unit": "%",
  "min": 0,
  "max": 100,
  "needle": true,
  "segments": [
    { "from": 0, "color": "#00FF00" },
    { "from": 50, "color": "#FFFF00" },
    { "from": 80, "color": "#FF0000" }
  ]
}
```

### Gauge Options

| Option | Type | Description |
|--------|------|-------------|
| `entity` | string | Entity ID (required) |
| `name` | string | Override name |
| `unit` | string | Unit of measurement |
| `min` | number | Minimum value (default: 0) |
| `max` | number | Maximum value (default: 100) |
| `needle` | boolean | Show needle style (required for segments) |
| `segments` | array | Color ranges with `from` and `color` |
| `severity` | object | Simple thresholds: `{ "green": 0, "yellow": 50, "red": 80 }` |

---

## History Graph Card

Historical data visualization.

```json
{
  "type": "history-graph",
  "title": "Temperature History",
  "hours_to_show": 24,
  "show_names": true,
  "entities": [
    { "entity": "sensor.indoor_temperature", "name": "Indoor" },
    { "entity": "sensor.outdoor_temperature", "name": "Outdoor" }
  ]
}
```

### Options

| Option | Type | Description |
|--------|------|-------------|
| `hours_to_show` | number | Time range (default: 24) |
| `show_names` | boolean | Show entity names |
| `logarithmic_scale` | boolean | Use log scale |
| `min_y_axis` | number | Minimum Y value |
| `max_y_axis` | number | Maximum Y value |

---

## Button Card

Trigger actions.

```json
{
  "type": "button",
  "name": "Turn Off All Lights",
  "icon": "mdi:lightbulb-off",
  "tap_action": {
    "action": "call-service",
    "service": "light.turn_off",
    "target": { "entity_id": "all" }
  }
}
```

---

## Markdown Card

Rich text display with templates.

```json
{
  "type": "markdown",
  "title": "Welcome",
  "content": "# Hello!\n\nCurrent temperature: {{ states('sensor.temperature') }}°F",
  "text_only": false
}
```

---

## Grid Card

Nested grid of cards.

```json
{
  "type": "grid",
  "columns": 2,
  "square": true,
  "cards": [
    { "type": "tile", "entity": "light.one" },
    { "type": "tile", "entity": "light.two" }
  ]
}
```

---

## Vertical Stack Card

Stack cards vertically.

```json
{
  "type": "vertical-stack",
  "cards": [
    { "type": "tile", "entity": "light.one" },
    { "type": "tile", "entity": "light.two" }
  ]
}
```

---

## Horizontal Stack Card

Stack cards horizontally.

```json
{
  "type": "horizontal-stack",
  "cards": [
    { "type": "tile", "entity": "light.one" },
    { "type": "tile", "entity": "light.two" }
  ]
}
```

---

## Conditional Card

Show card based on conditions.

```json
{
  "type": "conditional",
  "conditions": [
    { "entity": "input_boolean.show_card", "state": "on" }
  ],
  "card": {
    "type": "tile",
    "entity": "light.living_room"
  }
}
```

---

## Thermostat Card

Climate control display.

```json
{
  "type": "thermostat",
  "entity": "climate.living_room",
  "features": [
    { "type": "climate-hvac-modes" },
    { "type": "climate-preset-modes" }
  ]
}
```

---

## Weather Forecast Card

```json
{
  "type": "weather-forecast",
  "entity": "weather.home",
  "show_forecast": true,
  "forecast_type": "daily"
}
```

---

## Map Card

```json
{
  "type": "map",
  "entities": ["device_tracker.phone", "zone.home"],
  "hours_to_show": 24,
  "default_zoom": 15
}
```

---

## Media Control Card

```json
{
  "type": "media-control",
  "entity": "media_player.living_room"
}
```

---

## Picture Elements Card

Image with interactive overlays.

```json
{
  "type": "picture-elements",
  "image": "/local/floorplan.png",
  "elements": [
    {
      "type": "state-icon",
      "entity": "light.living_room",
      "style": { "left": "50%", "top": "30%" },
      "tap_action": { "action": "toggle" }
    }
  ]
}
```

---

## Alarm Panel Card

```json
{
  "type": "alarm-panel",
  "entity": "alarm_control_panel.home",
  "states": ["arm_home", "arm_away", "arm_night"]
}
```

---

## Calendar Card

```json
{
  "type": "calendar",
  "entities": ["calendar.family"]
}
```

---

## Iframe Card

```json
{
  "type": "iframe",
  "url": "https://grafana.local/dashboard",
  "aspect_ratio": "16:9"
}
```

---

## Entity Card

Single entity display (simpler than tile).

```json
{
  "type": "entity",
  "entity": "sensor.temperature",
  "name": "Current Temperature",
  "icon": "mdi:thermometer",
  "unit": "°F"
}
```

---

## Actions Reference

Actions define what happens on tap/hold/double-tap.

| Action | Description |
|--------|-------------|
| `more-info` | Show entity details dialog |
| `toggle` | Toggle entity state |
| `call-service` | Call a Home Assistant service |
| `navigate` | Navigate to another view/dashboard |
| `url` | Open external URL |
| `none` | No action |

### Call Service Example

```json
{
  "tap_action": {
    "action": "call-service",
    "service": "light.turn_on",
    "data": { "brightness_pct": 100 },
    "target": { "entity_id": "light.living_room" }
  }
}
```

### Navigate Example

```json
{
  "tap_action": {
    "action": "navigate",
    "navigation_path": "/lovelace/settings"
  }
}
```
