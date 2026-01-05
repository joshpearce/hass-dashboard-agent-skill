# Troubleshooting

Common errors and solutions for Home Assistant dashboard creation.

## Chrome DevTools MCP Server Not Connected

**Error**: `No such tool available: mcp__chrome-devtools__*`

**Cause**: The Chrome DevTools MCP server is not configured.

**Solution**: Install the MCP server:

```bash
claude mcp add chrome-devtools npx chrome-devtools-mcp@latest
```

After adding, restart Claude Code to load the new configuration.

---

## "Url path needs to contain a hyphen" Error

**Cause**: The `url_path` parameter must contain at least one hyphen.

**Solution**: Use kebab-case with hyphens:
- `lights` → `all-lights`
- `power` → `power-monitoring`
- `temps` → `temp-sensors`

```javascript
// Wrong
url_path: 'lights'

// Correct
url_path: 'all-lights'
```

---

## "Unknown config specified" Error

**Cause**: Trying to save configuration to a dashboard that doesn't exist yet.

**Solution**: Call `lovelace/dashboards/create` before `lovelace/config/save`:

```javascript
// Step 1: Create the dashboard first
await hass.callWS({
  type: 'lovelace/dashboards/create',
  url_path: 'my-dashboard',
  title: 'My Dashboard',
  icon: 'mdi:view-dashboard',
  show_in_sidebar: true,
  require_admin: false
});

// Step 2: Then save configuration
await hass.callWS({
  type: 'lovelace/config/save',
  url_path: 'my-dashboard',
  config: { views: [...] }
});
```

---

## Dashboard Not Appearing After Creation

**Possible causes**:
1. `lovelace/dashboards/create` failed silently
2. `url_path` mismatch between create and save
3. Browser cache

**Solutions**:
1. Check for errors in the websocket call return value
2. Ensure `url_path` matches exactly in both calls
3. Hard refresh browser (Ctrl+Shift+R / Cmd+Shift+R)

```javascript
// Check for errors
const result = await hass.callWS({
  type: 'lovelace/dashboards/create',
  url_path: 'my-dashboard',
  ...
});
console.log('Create result:', result);  // Check for errors
```

---

## Dashboard Already Exists Error

**Cause**: Trying to create a dashboard with a `url_path` that already exists.

**Solution**: Either use a different `url_path`, or just update the existing dashboard:

```javascript
// Option 1: Use different url_path
url_path: 'my-dashboard-v2'

// Option 2: Update existing (skip create, just save)
await hass.callWS({
  type: 'lovelace/config/save',
  url_path: 'existing-dashboard',
  config: { views: [...] }
});
```

---

## Cards Not Rendering

**Possible causes**:
1. Invalid entity IDs
2. Misspelled card type
3. Missing required fields

**Solutions**:

Check entity exists:
```javascript
const state = hass.states['light.living_room'];
if (!state) {
  console.error('Entity not found');
}
```

Verify card type spelling:
```javascript
// Wrong
{ type: 'tiles', entity: '...' }

// Correct
{ type: 'tile', entity: '...' }
```

Ensure required fields:
```javascript
// Wrong - missing entity
{ type: 'tile', name: 'My Light' }

// Correct
{ type: 'tile', entity: 'light.living_room', name: 'My Light' }
```

---

## Changes Not Reflected

**Cause**: Browser caching or memory state not refreshed.

**Solutions**:
1. Hard refresh: Ctrl+Shift+R (Windows/Linux) or Cmd+Shift+R (Mac)
2. Navigate away and back to the dashboard
3. Clear browser cache

For websocket API changes, they should apply immediately after page refresh.

---

## File Changes Not Working

**Cause**: Direct file editing doesn't update running Home Assistant.

**Problem**: Editing files in `/config/.storage/` directly:
- Dashboard registry changes (`lovelace_dashboards`) require HA restart
- Dashboard content changes may not be picked up

**Solution**: Always use the websocket API:
```javascript
// Use this (immediate effect)
await hass.callWS({
  type: 'lovelace/config/save',
  url_path: 'my-dashboard',
  config: {...}
});

// Not this (requires restart)
// Editing /config/.storage/lovelace.my_dashboard directly
```

---

## hass Object Not Found

**Cause**: Script running before Home Assistant frontend fully loaded.

**Solution**: Wait for the hass object:

```javascript
async () => {
  // Wait for hass to be available
  let hass = document.querySelector('home-assistant')?.hass;
  let attempts = 0;
  while (!hass && attempts < 10) {
    await new Promise(r => setTimeout(r, 500));
    hass = document.querySelector('home-assistant')?.hass;
    attempts++;
  }

  if (!hass) {
    return { error: 'hass object not available - is HA logged in?' };
  }

  // Continue with operations...
}
```

---

## Entity Has No Area

**Cause**: Entity not assigned to an area directly, and its device may also lack area assignment.

**Solution**: Check both entity and device for area:

```javascript
const entities = await hass.callWS({ type: 'config/entity_registry/list' });
const devices = await hass.callWS({ type: 'config/device_registry/list' });
const areas = await hass.callWS({ type: 'config/area_registry/list' });

const areaLookup = Object.fromEntries(areas.map(a => [a.area_id, a.name]));
const deviceLookup = Object.fromEntries(devices.map(d => [d.id, d.area_id]));

function getEntityArea(entity) {
  // Check entity's direct area assignment
  if (entity.area_id) {
    return areaLookup[entity.area_id] || 'Unknown Area';
  }
  // Fall back to device's area
  if (entity.device_id && deviceLookup[entity.device_id]) {
    return areaLookup[deviceLookup[entity.device_id]] || 'Unknown Area';
  }
  return 'Unassigned';
}
```

---

## Sections Layout Has Too Much Whitespace

**Cause**: Sections view auto-arranges with responsive spacing.

**Solution**: Use panel view with stacks for dense layouts:

```javascript
// Instead of sections
{
  type: 'sections',
  sections: [...]
}

// Use panel with stacks
{
  type: 'panel',
  cards: [{
    type: 'vertical-stack',
    cards: [
      { type: 'horizontal-stack', cards: [...] },
      { type: 'horizontal-stack', cards: [...] }
    ]
  }]
}
```

---

## Tiles Too Large

**Solutions**:

1. Use `vertical: true` for compact display:
```json
{
  "type": "tile",
  "entity": "light.example",
  "vertical": true
}
```

2. Use `grid_options` to control size (in sections view):
```json
{
  "type": "tile",
  "entity": "light.example",
  "grid_options": { "columns": 3, "rows": 1 }
}
```

3. Use `hide_state` to remove state text:
```json
{
  "type": "tile",
  "entity": "light.example",
  "hide_state": true
}
```

---

## Debugging Tips

### List All Dashboards
```javascript
const dashboards = await hass.callWS({ type: 'lovelace/dashboards/list' });
console.log(dashboards);
```

### Get Dashboard Configuration
```javascript
const config = await hass.callWS({
  type: 'lovelace/config',
  url_path: 'my-dashboard'
});
console.log(JSON.stringify(config, null, 2));
```

### List All Entities
```javascript
console.log(Object.keys(hass.states).sort());
```

### Check Entity State
```javascript
const entity = hass.states['light.living_room'];
console.log('State:', entity.state);
console.log('Attributes:', entity.attributes);
```

### Find Entities by Domain
```javascript
const lights = Object.keys(hass.states)
  .filter(id => id.startsWith('light.'));
console.log(lights);
```
