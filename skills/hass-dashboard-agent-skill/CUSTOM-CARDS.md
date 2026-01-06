# Custom JavaScript Cards

Create custom Lovelace cards as JavaScript Web Components for full control over layout and styling.

## When to Use Custom Cards

| Approach | Use When |
|----------|----------|
| Native cards | Standard layouts (entities, tiles, gauges) |
| Markdown + Jinja2 | Dynamic text content, simple tables |
| Custom JS card | Complex tables, custom layouts, full CSS control |

Custom cards are ideal when:
- You need HTML class attributes (markdown sanitizes them)
- You want precise table/grid layouts
- You need interactive elements beyond native cards
- Theme consistency with full CSS control is required

---

## Basic Structure

Custom cards are [Web Components](https://developer.mozilla.org/en-US/docs/Web/API/Web_components) that extend `HTMLElement`:

```javascript
class MyCustomCard extends HTMLElement {
  // Called when card config is set
  setConfig(config) {
    if (!config.required_field) {
      throw new Error("Please define required_field");
    }
    this.config = config;
  }

  // Called when hass state updates (frequently)
  set hass(hass) {
    this._hass = hass;
    this.render();
  }

  // Render the card content
  render() {
    if (!this._hass || !this.config) return;

    this.innerHTML = `
      <ha-card>
        <style>/* CSS here */</style>
        <div class="content">
          <!-- HTML here -->
        </div>
      </ha-card>
    `;
  }

  // Return card height in units (1 unit = 50px)
  getCardSize() {
    return 3;
  }
}

// Register the custom element
customElements.define("my-custom-card", MyCustomCard);
```

---

## Native CSS Variables

**Always use Home Assistant's CSS variables** for theme consistency. The card will automatically adapt to any theme.

### Colors

```css
/* Primary theme colors */
var(--primary-color)              /* Main accent (cyan/blue) */
var(--accent-color)               /* Secondary accent */
var(--primary-text-color)         /* Main text */
var(--secondary-text-color)       /* Muted text */

/* Backgrounds */
var(--card-background-color)      /* Card surface */
var(--secondary-background-color) /* Subtle contrast */
var(--primary-background-color)   /* Page background */

/* State colors */
var(--success-color)              /* Green - OK states */
var(--warning-color)              /* Yellow/orange - warnings */
var(--error-color)                /* Red - errors/alerts */
var(--info-color)                 /* Blue - informational */

/* Borders */
var(--divider-color)              /* Subtle dividers */
var(--outline-color)              /* Focus outlines */
```

### Typography

```css
var(--ha-font-family-body)        /* Roboto, system fonts */
var(--ha-font-family-heading)     /* Headings */

/* Sizes */
var(--ha-font-size-xs)            /* 10px - labels */
var(--ha-font-size-s)             /* 12px - small */
var(--ha-font-size-m)             /* 14px - body */
var(--ha-font-size-l)             /* 16px - large */
var(--ha-font-size-xl)            /* 20px - titles */

/* Weights */
var(--ha-font-weight-normal)      /* 400 */
var(--ha-font-weight-medium)      /* 500 */
var(--ha-font-weight-bold)        /* 700 */
```

### Spacing & Borders

```css
/* Border radius */
var(--ha-border-radius-sm)        /* Small (4px) */
var(--ha-border-radius)           /* Default (8px) */
var(--ha-border-radius-lg)        /* Large (12px) */

/* Border widths */
var(--ha-border-width-sm)         /* 1px */
var(--ha-border-width-md)         /* 2px */
```

---

## Reusable Card: entity-grid-card

A generic, config-driven grid/table card included with this skill. Use this instead of building specialized cards.

**Location:** `/config/www/entity-grid-card.js`

### Configuration

```json
{
  "type": "custom:entity-grid-card",
  "title": "Smoke Detector Status",
  "columns": [
    { "header": "Location", "field": "name" },
    { "header": "Smoke", "field": "smoke", "render": "state_icon" },
    { "header": "Battery", "field": "battery", "render": "battery" }
  ],
  "rows": [
    {
      "name": "Garage",
      "smoke": "binary_sensor.garage_smoke",
      "battery": "sensor.garage_battery"
    }
  ]
}
```

### Column Options

| Property | Description |
|----------|-------------|
| `header` | Column header text |
| `field` | Key in row data (use `name` for row label) |
| `render` | Renderer type (see below) |
| `align` | `left`, `center` (default), or `right` |

### Built-in Renderers

| Renderer | Description | Output |
|----------|-------------|--------|
| `text` | Raw state value | Plain text |
| `state_icon` | Binary alert status | Alert (red) / Clear (green) |
| `state_icon_inverted` | Inverted (off=alert) | Clear / Alert |
| `state_ok` | OK/problem status | OK (green) / Yes (red) |
| `battery` | Percentage with color | 99.0% (green/yellow/red) |
| `percentage` | Simple percentage | 85.0% |
| `toggle` | On/off toggle | On (gray) / Off (gray) |
| `node_status` | Z-Wave node status | Awake (green) / Asleep (gray) |
| `number` | Numeric with 1 decimal | 23.5 |

### Custom Renderer Config

Override default text/thresholds:

```json
{
  "renderers": {
    "state_icon": { "on": "ALERT", "off": "OK", "unavailable": "N/A" },
    "battery": { "thresholds": [20, 50] },
    "toggle": { "on": "Muted", "off": "Active" },
    "node_status": { "awake": "Online", "asleep": "Sleep", "dead": "Dead" }
  }
}
```

---

## Accessing Entity State

```javascript
// Get entity state
getState(entityId) {
  return this._hass?.states[entityId]?.state || "unknown";
}

// Get entity attribute
getAttribute(entityId, attr) {
  return this._hass?.states[entityId]?.attributes?.[attr];
}

// Get friendly name
getName(entityId) {
  return this._hass?.states[entityId]?.attributes?.friendly_name || entityId;
}
```

---

## Complete Example: Status Table Card

```javascript
class StatusTableCard extends HTMLElement {
  setConfig(config) {
    if (!config.entities) {
      throw new Error("Please define entities");
    }
    this.config = config;
  }

  set hass(hass) {
    this._hass = hass;
    this.render();
  }

  getState(entityId) {
    return this._hass?.states[entityId]?.state || "unknown";
  }

  getStatusIcon(state) {
    if (state === "on" || state === "home") {
      return `<span class="status ok">✅</span>`;
    } else if (state === "off" || state === "not_home") {
      return `<span class="status off">⚪</span>`;
    } else if (state === "unavailable") {
      return `<span class="status unavailable">➖</span>`;
    }
    return `<span class="status unknown">❓</span>`;
  }

  render() {
    if (!this._hass || !this.config) return;

    const rows = this.config.entities.map(e => {
      const name = this._hass.states[e]?.attributes?.friendly_name || e;
      const state = this.getState(e);
      return `
        <tr>
          <td class="name">${name}</td>
          <td class="state">${this.getStatusIcon(state)}</td>
        </tr>
      `;
    }).join("");

    this.innerHTML = `
      <ha-card>
        <style>
          .card-title {
            padding: 12px 16px;
            font-family: var(--ha-font-family-body);
            font-size: var(--ha-font-size-xl);
            color: var(--primary-text-color);
          }
          .status-table {
            width: 100%;
            border-collapse: collapse;
            font-family: var(--ha-font-family-body);
            font-size: var(--ha-font-size-m);
          }
          .status-table th {
            background: var(--secondary-background-color);
            color: var(--primary-color);
            padding: 10px;
            text-transform: uppercase;
            font-size: var(--ha-font-size-xs);
            border-bottom: var(--ha-border-width-md) solid var(--primary-color);
          }
          .status-table td {
            padding: 10px;
            border-bottom: var(--ha-border-width-sm) solid var(--divider-color);
            color: var(--primary-text-color);
          }
          .status-table tr:hover {
            background: var(--secondary-background-color);
          }
          .status.ok { color: var(--success-color); }
          .status.off { color: var(--secondary-text-color); }
          .status.unavailable { color: var(--disabled-text-color); }
        </style>
        ${this.config.title ? `<div class="card-title">${this.config.title}</div>` : ""}
        <table class="status-table">
          <thead>
            <tr>
              <th>Name</th>
              <th>Status</th>
            </tr>
          </thead>
          <tbody>${rows}</tbody>
        </table>
      </ha-card>
    `;
  }

  getCardSize() {
    return Math.ceil((this.config?.entities?.length || 1) / 2) + 1;
  }
}

customElements.define("status-table-card", StatusTableCard);
```

---

## Deploying Custom Cards

### Step 1: Create the JS File

Save your card to `/config/www/my-card.js` on the Home Assistant server.

### Step 2: Register as Resource

```javascript
async () => {
  const hass = document.querySelector('home-assistant')?.hass;

  await hass.callWS({
    type: 'lovelace/resources/create',
    url: `/local/my-card.js?v=${Date.now()}`,
    res_type: 'module'
  });

  return { success: true };
}
```

**Note:** The `?v=${Date.now()}` query parameter busts the browser cache when updating the file.

### Step 3: Use in Dashboard

```javascript
{
  type: 'custom:my-card',
  title: 'My Card',
  entities: ['sensor.one', 'sensor.two']
}
```

---

## Updating a Custom Card

When updating the JS file, you must bust the browser cache:

```javascript
async () => {
  const hass = document.querySelector('home-assistant')?.hass;
  const resources = await hass.callWS({ type: 'lovelace/resources' });

  const resource = resources.find(r => r.url.includes('my-card.js'));
  if (resource) {
    await hass.callWS({
      type: 'lovelace/resources/update',
      resource_id: resource.id,
      url: `/local/my-card.js?v=${Date.now()}`,
      res_type: 'module'
    });
  }

  return { success: true };
}
```

Then reload the browser or navigate away and back.

---

## Listing Registered Resources

```javascript
async () => {
  const hass = document.querySelector('home-assistant')?.hass;
  return await hass.callWS({ type: 'lovelace/resources' });
}
```

---

## Best Practices

1. **Always use CSS variables** - Never hardcode colors; use `var(--primary-color)` etc.
2. **Wrap in `<ha-card>`** - Provides consistent card styling and shadow
3. **Handle missing states** - Check for `unavailable` and `unknown` states
4. **Provide fallbacks** - Use `var(--primary-color, #009ac7)` syntax
5. **Keep renders efficient** - The `set hass()` setter is called frequently
6. **Use cache busting** - Always update resource URL when changing JS files
