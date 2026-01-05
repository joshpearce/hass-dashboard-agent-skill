---
name: hass-dashboard-agent-skill
description: Creates and modifies Home Assistant Lovelace dashboards using the websocket API via Chrome DevTools. Use when the user wants to create, update, or delete Home Assistant dashboards, add cards or views, or query entities for dashboard building.
allowed-tools: Read, Glob, Grep, mcp__chrome-devtools__*
---

# Home Assistant Dashboard Creation Skill

Creates and modifies Home Assistant Lovelace dashboards using the websocket API via Chrome DevTools.

## Quick Reference

- [CARDS.md](CARDS.md) - Card types reference (tile, entities, gauge, etc.)
- [VIEWS.md](VIEWS.md) - View types and layout strategies
- [EXAMPLES.md](EXAMPLES.md) - Complete dashboard examples
- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) - Common errors and solutions

## Workflow

### Step 1: Get Home Assistant URL

**Before doing anything else**, prompt the user for their Home Assistant URL. Do not assume or guess the URL.

Example prompt: "What is your Home Assistant URL? (e.g., https://homeassistant.local:8123 or https://ha.example.com)"

### Step 2: Connect via Chrome DevTools

```javascript
// Open the page
mcp__chrome-devtools__new_page({ url: "<user's HA URL>" })

// Verify login
mcp__chrome-devtools__take_snapshot()

// Access the hass object
const hass = document.querySelector('home-assistant')?.hass;
```

### Step 3: Query Entities (if needed)

```javascript
async () => {
  const hass = document.querySelector('home-assistant')?.hass;

  const entities = await hass.callWS({ type: 'config/entity_registry/list' });
  const devices = await hass.callWS({ type: 'config/device_registry/list' });
  const areas = await hass.callWS({ type: 'config/area_registry/list' });

  // Create lookups
  const areaLookup = Object.fromEntries(areas.map(a => [a.area_id, a.name]));
  const deviceLookup = Object.fromEntries(devices.map(d => [d.id, {
    name: d.name_by_user || d.name,
    area_id: d.area_id
  }]));

  // Example: Find all lights with area info
  const lights = entities
    .filter(e => e.entity_id.startsWith('light.') && !e.disabled_by)
    .map(e => {
      const state = hass.states[e.entity_id];
      let areaName = 'Unassigned';
      if (e.area_id) {
        areaName = areaLookup[e.area_id] || 'Unassigned';
      } else if (e.device_id && deviceLookup[e.device_id]) {
        areaName = areaLookup[deviceLookup[e.device_id].area_id] || 'Unassigned';
      }
      return {
        entity_id: e.entity_id,
        name: state?.attributes?.friendly_name || e.entity_id,
        area: areaName,
        state: state?.state
      };
    });

  return lights;
}
```

### Step 4: Create Dashboard

**CRITICAL**: The `url_path` MUST contain a hyphen (`-`). Single words like `lights` will fail.

```javascript
async () => {
  const hass = document.querySelector('home-assistant')?.hass;

  // Step 1: Create dashboard in registry
  await hass.callWS({
    type: 'lovelace/dashboards/create',
    url_path: 'my-dashboard',        // MUST contain hyphen
    title: 'My Dashboard',
    icon: 'mdi:view-dashboard',
    show_in_sidebar: true,
    require_admin: false
  });

  // Step 2: Save configuration
  await hass.callWS({
    type: 'lovelace/config/save',
    url_path: 'my-dashboard',
    config: {
      views: [{
        title: 'Home',
        type: 'sections',  // or 'panel' for dense layouts
        sections: [{
          type: 'grid',
          cards: [
            { type: 'heading', heading: 'My Section' },
            { type: 'tile', entity: 'light.example' }
          ]
        }]
      }]
    }
  });

  return { success: true };
}
```

### Step 5: Update Existing Dashboard

For existing dashboards, use `lovelace/config/save` directly - no need to recreate:

```javascript
await hass.callWS({
  type: 'lovelace/config/save',
  url_path: 'existing-dashboard',
  config: { /* new configuration */ }
});
```

### Step 6: Delete Dashboard

```javascript
async () => {
  const hass = document.querySelector('home-assistant')?.hass;

  // List dashboards to find the ID
  const dashboards = await hass.callWS({ type: 'lovelace/dashboards/list' });

  // Delete by ID (not url_path)
  await hass.callWS({
    type: 'lovelace/dashboards/delete',
    dashboard_id: 'the_dashboard_id'
  });
}
```

## Layout Strategy

| Use Case | View Type | Structure |
|----------|-----------|-----------|
| Standard dashboard | `sections` | Auto-arranged grid sections |
| Dense/process control | `panel` | Nested horizontal/vertical stacks |
| Mixed card sizes | `masonry` | Flowing column layout |

**Dense layout pattern** (panel + stacks):

```javascript
{
  type: 'panel',
  cards: [{
    type: 'vertical-stack',
    cards: [
      {
        type: 'horizontal-stack',
        cards: [/* row 1 */]
      },
      {
        type: 'horizontal-stack',
        cards: [/* row 2 */]
      }
    ]
  }]
}
```

## Common Card Types

| Card | Use For |
|------|---------|
| `tile` | Individual entities with optional controls |
| `entities` | Dense lists of related items |
| `heading` | Section labels |
| `gauge` | Numeric sensors with visual range |
| `history-graph` | Time series data |
| `button` | Triggering actions/scripts |

See [CARDS.md](CARDS.md) for complete reference.

## Entity Domains

| Domain | Description |
|--------|-------------|
| `light` | Lighting devices |
| `switch` | On/off devices |
| `sensor` | Measurements (temp, power, etc.) |
| `binary_sensor` | Two-state (motion, door, etc.) |
| `climate` | Thermostats, HVAC |
| `cover` | Garage doors, blinds |
| `media_player` | TVs, speakers |

## Best Practices

- **URL paths**: Must use kebab-case with hyphen (`all-lights`, not `lights`)
- **Dense layouts**: Use `panel` view with `horizontal-stack`/`vertical-stack`
- **Compact tiles**: Set `vertical: true` for icon-centric display
- **Performance**: Limit `hours_to_show` on history graphs
- **Always query first**: Use entity/device/area registries before building dashboards dynamically
