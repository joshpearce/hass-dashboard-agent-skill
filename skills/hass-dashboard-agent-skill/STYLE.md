# Style Guide

User preferences and best practices for dashboard layout and visual design.

## Card Selection for Grouped Entities

### Preferred: Entities Card with Title

When displaying multiple entities grouped by category (area, device type, etc.), use the `entities` card with a `title`. This provides clear visual hierarchy with the group name as a card header and entities listed below.

```json
{
  "type": "entities",
  "title": "Master Bedroom",
  "state_color": true,
  "entities": [
    { "entity": "light.nightstand", "name": "Nightstand" },
    { "entity": "light.ceiling", "name": "Ceiling Light" },
    { "entity": "light.fan_light", "name": "Fan Light" }
  ]
}
```

**Benefits:**
- Clear visual hierarchy (title is part of the card, not a peer element)
- Compact list layout
- Built-in header toggle for controlling all entities
- Consistent with other dashboards (e.g., Alarms)

**Trade-off:** Adjusting brightness or other controls requires tapping to open the more-info dialog.

### Alternative: Tile Cards with Inline Controls

When minimizing clicks is the priority (e.g., frequent brightness adjustments), use `tile` cards with `features`:

```json
{
  "type": "tile",
  "entity": "light.living_room",
  "name": "Living Room",
  "features": [
    { "type": "light-brightness" }
  ]
}
```

**Benefits:**
- Brightness slider visible without tapping
- Larger touch targets

**Trade-off:** Harder to create visual hierarchy; area labels appear as peer elements rather than containers.

---

## Layout Selection

| Use Case | View Type | Structure |
|----------|-----------|-----------|
| Standard dashboard | `masonry` | Entities cards flow into columns |
| Dense/process control | `panel` | Nested horizontal/vertical stacks |
| Auto-arranged sections | `sections` | Grid sections (more whitespace) |

### Recommended: Masonry with Entities Cards

For most dashboards, use `masonry` view with `entities` cards grouped by area:

```json
{
  "views": [{
    "title": "All Lights",
    "type": "masonry",
    "cards": [
      {
        "type": "entities",
        "title": "Master",
        "state_color": true,
        "entities": [...]
      },
      {
        "type": "entities",
        "title": "Office",
        "state_color": true,
        "entities": [...]
      }
    ]
  }]
}
```

This provides:
- Compact layout (cards flow into columns)
- Clear hierarchy (area names as card titles)
- Minimal whitespace

### Avoid: Sections View for Dense Layouts

The `sections` view creates distinct grid sections with significant whitespace between them. Use `masonry` instead for more compact results.

### Avoid: Markdown/Heading as Peer Labels

Using `markdown` or `heading` cards for area labels creates peer elements rather than true hierarchy:

```json
// NOT RECOMMENDED - label is a peer, not a container
{
  "type": "vertical-stack",
  "cards": [
    { "type": "markdown", "content": "## Master" },
    { "type": "tile", "entity": "light.one" },
    { "type": "tile", "entity": "light.two" }
  ]
}
```

---

## Entity Row Options

For entities within an `entities` card:

| Option | Use For |
|--------|---------|
| `secondary_info: brightness` | Show current brightness % for dimmable lights |
| `secondary_info: last-changed` | Show when entity last changed state |
| `state_color: true` | Color icons based on state (yellow when on) |

Example with secondary info:

```json
{
  "type": "entities",
  "title": "Office",
  "state_color": true,
  "entities": [
    { "entity": "light.desk_lamp", "name": "Desk Lamp", "secondary_info": "brightness" },
    { "entity": "light.overhead", "name": "Overhead", "secondary_info": "brightness" },
    { "entity": "switch.fan", "name": "Fan" }
  ]
}
```

---

## Dense Layout Pattern (Process Control)

For maximum density with precise control, use `panel` view with nested stacks:

```json
{
  "type": "panel",
  "cards": [{
    "type": "vertical-stack",
    "cards": [
      {
        "type": "horizontal-stack",
        "cards": [
          { "type": "entities", "title": "Area 1", "entities": [...] },
          { "type": "entities", "title": "Area 2", "entities": [...] }
        ]
      },
      {
        "type": "horizontal-stack",
        "cards": [
          { "type": "entities", "title": "Area 3", "entities": [...] },
          { "type": "entities", "title": "Area 4", "entities": [...] }
        ]
      }
    ]
  }]
}
```

---

## Color Guidelines

- Use `state_color: true` on entities cards to indicate on/off states
- For tile cards, `amber` is the default light color
- Available colors: `primary`, `accent`, `red`, `pink`, `purple`, `deep-purple`, `indigo`, `blue`, `light-blue`, `cyan`, `teal`, `green`, `light-green`, `lime`, `yellow`, `amber`, `orange`, `deep-orange`, `brown`, `grey`, `blue-grey`, `black`, `white`
