# Views and Layout Reference

Views are the top-level pages/tabs within a dashboard. Each dashboard can have multiple views.

## View Configuration

```json
{
  "title": "View Title",
  "path": "url-path",
  "icon": "mdi:icon-name",
  "type": "sections",
  "sections": [],
  "cards": [],
  "header": {},
  "theme": "theme_name",
  "visible": true,
  "subview": false,
  "background": {}
}
```

## View Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `title` | string | Yes | View tab name |
| `path` | string | No | URL path segment |
| `icon` | string | No | Tab icon (MDI format) |
| `type` | string | No | Layout type (see below) |
| `sections` | array | No | For sections layout |
| `cards` | array | No | For other layouts |
| `header` | object | No | View header card configuration |
| `theme` | string | No | Theme override |
| `visible` | boolean/array | No | Visibility control |
| `subview` | boolean | No | Mark as subview (no tab) |
| `back_path` | string | No | Navigation path for subviews |
| `background` | object | No | Background image configuration |

---

## View Types

| Type | Description | Best For |
|------|-------------|----------|
| `sections` | Grid-based layout with auto-arranged sections | Standard dashboards, responsive layouts |
| `panel` | Single card fills the view | Dense/process control layouts (use with stacks) |
| `masonry` | Cards arranged in columns by size | Mixed card sizes, flowing layouts |
| `sidebar` | Two columns (wide + narrow sidebar) | Main content with secondary info |

---

## Sections Layout

The default and recommended layout for most dashboards. Sections auto-arrange in a responsive grid.

```json
{
  "type": "sections",
  "sections": [
    {
      "type": "grid",
      "cards": [
        { "type": "heading", "heading": "Section 1" },
        { "type": "tile", "entity": "light.one" }
      ]
    },
    {
      "type": "grid",
      "cards": [
        { "type": "heading", "heading": "Section 2" },
        { "type": "tile", "entity": "light.two" }
      ]
    }
  ]
}
```

### Section Structure

Each section is a grid container:

```json
{
  "type": "grid",
  "cards": []
}
```

### Controlling Columns

Use `max_columns` on the view to limit section columns:

```json
{
  "type": "sections",
  "max_columns": 4,
  "sections": [...]
}
```

---

## Panel Layout (Dense/Process Control)

For maximum layout control, use `panel` with nested stacks. The single card fills the entire view area.

```json
{
  "type": "panel",
  "cards": [{
    "type": "vertical-stack",
    "cards": [
      {
        "type": "horizontal-stack",
        "cards": [
          /* Row 1: Area cards side by side */
        ]
      },
      {
        "type": "horizontal-stack",
        "cards": [
          /* Row 2: More area cards */
        ]
      }
    ]
  }]
}
```

### Dense Layout Pattern

For process control layouts with maximum density, see [STYLE.md](STYLE.md#dense-layout-pattern-process-control).

---

## Masonry Layout

Cards flow into columns based on their size. Good for dashboards with mixed card types.

```json
{
  "type": "masonry",
  "cards": [
    { "type": "weather-forecast", "entity": "weather.home" },
    { "type": "tile", "entity": "light.one" },
    { "type": "history-graph", "entities": [...] },
    { "type": "tile", "entity": "light.two" }
  ]
}
```

---

## Sidebar Layout

Two-column layout with main content and sidebar.

```json
{
  "type": "sidebar",
  "cards": [
    /* Main content cards */
  ],
  "sidebar": [
    /* Sidebar cards */
  ]
}
```

---

## View Header

Add a header card at the top of a view:

```json
{
  "header": {
    "card": {
      "type": "markdown",
      "text_only": true,
      "content": "# Dashboard Title\nSubtitle or description"
    }
  }
}
```

---

## View Background

```json
{
  "background": {
    "image": "/local/images/background.jpg",
    "opacity": 50,
    "size": "cover",
    "alignment": "center",
    "attachment": "fixed"
  }
}
```

---

## Subviews

Subviews don't appear as tabs - they're navigated to via actions.

```json
{
  "title": "Device Details",
  "path": "device-details",
  "subview": true,
  "back_path": "/lovelace/home",
  "cards": [...]
}
```

Navigate to subview:

```json
{
  "tap_action": {
    "action": "navigate",
    "navigation_path": "/lovelace/device-details"
  }
}
```

---

## Layout Comparison

| Feature | sections | panel | masonry |
|---------|----------|-------|---------|
| Auto-responsive | Yes | No | Yes |
| Layout control | Limited | Full | Limited |
| Card arrangement | Grid sections | Single card + stacks | Flowing columns |
| Whitespace | More | Minimal | Moderate |

See [STYLE.md](STYLE.md) for layout recommendations and user preferences.
