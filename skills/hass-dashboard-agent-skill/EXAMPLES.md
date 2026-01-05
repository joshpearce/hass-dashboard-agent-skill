# Dashboard Examples

Complete examples for common dashboard patterns.

## Example 1: Temperature Dashboard (Sections Layout)

Standard responsive dashboard for temperature monitoring.

```javascript
async () => {
  const hass = document.querySelector('home-assistant')?.hass;

  // Create dashboard
  await hass.callWS({
    type: 'lovelace/dashboards/create',
    url_path: 'temperature-sensors',
    title: 'Temperature Sensors',
    icon: 'mdi:thermometer',
    show_in_sidebar: true,
    require_admin: false
  });

  // Save configuration
  await hass.callWS({
    type: 'lovelace/config/save',
    url_path: 'temperature-sensors',
    config: {
      views: [{
        title: 'All Temperatures',
        type: 'sections',
        header: {
          card: {
            type: 'markdown',
            text_only: true,
            content: '# Temperature Monitoring\nAll temperature sensors in the home'
          }
        },
        sections: [
          {
            type: 'grid',
            cards: [
              { type: 'heading', heading: 'Indoor' },
              {
                type: 'tile',
                entity: 'sensor.living_room_temperature',
                name: 'Living Room',
                color: 'orange'
              },
              {
                type: 'tile',
                entity: 'sensor.bedroom_temperature',
                name: 'Bedroom',
                color: 'orange'
              },
              {
                type: 'history-graph',
                title: '24 Hour History',
                hours_to_show: 24,
                entities: [
                  { entity: 'sensor.living_room_temperature', name: 'Living Room' },
                  { entity: 'sensor.bedroom_temperature', name: 'Bedroom' }
                ]
              }
            ]
          },
          {
            type: 'grid',
            cards: [
              { type: 'heading', heading: 'Outdoor' },
              {
                type: 'gauge',
                entity: 'sensor.outdoor_temperature',
                name: 'Current',
                min: -20,
                max: 120,
                needle: true,
                segments: [
                  { from: -20, color: '#0000FF' },
                  { from: 32, color: '#00FFFF' },
                  { from: 50, color: '#00FF00' },
                  { from: 70, color: '#FFFF00' },
                  { from: 85, color: '#FF8800' },
                  { from: 100, color: '#FF0000' }
                ]
              }
            ]
          }
        ]
      }]
    }
  });

  return { success: true };
}
```

---

## Example 2: Dense Lights Dashboard (Panel Layout)

Process control style with all lights organized by area.

```javascript
async () => {
  const hass = document.querySelector('home-assistant')?.hass;

  // Query lights with area info
  const entities = await hass.callWS({ type: 'config/entity_registry/list' });
  const devices = await hass.callWS({ type: 'config/device_registry/list' });
  const areas = await hass.callWS({ type: 'config/area_registry/list' });

  const areaLookup = Object.fromEntries(areas.map(a => [a.area_id, a.name]));
  const deviceLookup = Object.fromEntries(devices.map(d => [d.id, d.area_id]));

  // Group lights by area
  const lightsByArea = {};
  entities
    .filter(e => e.entity_id.startsWith('light.') && !e.disabled_by)
    .forEach(e => {
      const areaId = e.area_id || (e.device_id && deviceLookup[e.device_id]);
      const areaName = areaId ? areaLookup[areaId] : 'Unassigned';
      if (!lightsByArea[areaName]) lightsByArea[areaName] = [];
      const state = hass.states[e.entity_id];
      lightsByArea[areaName].push({
        entity_id: e.entity_id,
        name: state?.attributes?.friendly_name || e.entity_id
      });
    });

  // Build area card
  const buildAreaCard = (areaName, lights) => ({
    type: 'vertical-stack',
    cards: [
      { type: 'heading', heading: areaName, heading_style: 'subtitle' },
      {
        type: 'grid',
        columns: Math.min(lights.length, 4),
        square: false,
        cards: lights.map(l => ({
          type: 'tile',
          entity: l.entity_id,
          name: l.name.replace(/ Light$/, '').replace(/ Lamp$/, ''),
          color: 'amber',
          vertical: true,
          tap_action: { action: 'toggle' },
          hold_action: { action: 'more-info' }
        }))
      }
    ]
  });

  // Build rows from areas
  const areaNames = Object.keys(lightsByArea).sort();
  const row1 = areaNames.slice(0, 4);
  const row2 = areaNames.slice(4);

  // Create dashboard
  await hass.callWS({
    type: 'lovelace/dashboards/create',
    url_path: 'all-lights',
    title: 'Lights',
    icon: 'mdi:lightbulb-group',
    show_in_sidebar: true,
    require_admin: false
  });

  // Save configuration
  await hass.callWS({
    type: 'lovelace/config/save',
    url_path: 'all-lights',
    config: {
      views: [{
        title: 'All Lights',
        type: 'panel',
        cards: [{
          type: 'vertical-stack',
          cards: [
            {
              type: 'horizontal-stack',
              cards: row1.map(area => buildAreaCard(area, lightsByArea[area]))
            },
            row2.length > 0 ? {
              type: 'horizontal-stack',
              cards: row2.map(area => buildAreaCard(area, lightsByArea[area]))
            } : null
          ].filter(Boolean)
        }]
      }]
    }
  });

  return { success: true, areas: areaNames.length, lights: entities.length };
}
```

---

## Example 3: Climate Control Dashboard

```javascript
async () => {
  const hass = document.querySelector('home-assistant')?.hass;

  await hass.callWS({
    type: 'lovelace/dashboards/create',
    url_path: 'climate-control',
    title: 'Climate',
    icon: 'mdi:thermostat',
    show_in_sidebar: true,
    require_admin: false
  });

  await hass.callWS({
    type: 'lovelace/config/save',
    url_path: 'climate-control',
    config: {
      views: [{
        title: 'Climate',
        type: 'sections',
        sections: [
          {
            type: 'grid',
            cards: [
              { type: 'heading', heading: 'Thermostats' },
              {
                type: 'thermostat',
                entity: 'climate.living_room',
                features: [
                  { type: 'climate-hvac-modes' },
                  { type: 'climate-preset-modes' }
                ]
              },
              {
                type: 'thermostat',
                entity: 'climate.bedroom'
              }
            ]
          },
          {
            type: 'grid',
            cards: [
              { type: 'heading', heading: 'Temperature Sensors' },
              {
                type: 'tile',
                entity: 'sensor.living_room_temperature',
                color: 'orange'
              },
              {
                type: 'tile',
                entity: 'sensor.bedroom_temperature',
                color: 'orange'
              },
              {
                type: 'tile',
                entity: 'sensor.outdoor_temperature',
                color: 'blue'
              }
            ]
          },
          {
            type: 'grid',
            cards: [
              { type: 'heading', heading: 'Humidity' },
              {
                type: 'gauge',
                entity: 'sensor.living_room_humidity',
                name: 'Living Room',
                min: 0,
                max: 100,
                needle: true,
                segments: [
                  { from: 0, color: '#FF0000' },
                  { from: 30, color: '#00FF00' },
                  { from: 60, color: '#FF0000' }
                ]
              }
            ]
          }
        ]
      }]
    }
  });

  return { success: true };
}
```

---

## Example 4: Power Monitoring Dashboard

```javascript
async () => {
  const hass = document.querySelector('home-assistant')?.hass;

  await hass.callWS({
    type: 'lovelace/dashboards/create',
    url_path: 'power-monitoring',
    title: 'Power',
    icon: 'mdi:flash',
    show_in_sidebar: true,
    require_admin: false
  });

  await hass.callWS({
    type: 'lovelace/config/save',
    url_path: 'power-monitoring',
    config: {
      views: [{
        title: 'Power',
        type: 'sections',
        sections: [
          {
            type: 'grid',
            cards: [
              { type: 'heading', heading: 'Current Usage' },
              {
                type: 'gauge',
                entity: 'sensor.total_power',
                name: 'Total Power',
                unit: 'W',
                min: 0,
                max: 10000,
                needle: true,
                segments: [
                  { from: 0, color: '#00FF00' },
                  { from: 3000, color: '#FFFF00' },
                  { from: 6000, color: '#FF0000' }
                ]
              },
              {
                type: 'tile',
                entity: 'sensor.daily_energy',
                name: 'Today',
                color: 'amber'
              },
              {
                type: 'tile',
                entity: 'sensor.monthly_energy',
                name: 'This Month',
                color: 'amber'
              }
            ]
          },
          {
            type: 'grid',
            cards: [
              { type: 'heading', heading: 'History' },
              {
                type: 'history-graph',
                title: 'Power Usage (24h)',
                hours_to_show: 24,
                entities: [
                  { entity: 'sensor.total_power', name: 'Total' }
                ]
              }
            ]
          },
          {
            type: 'grid',
            cards: [
              { type: 'heading', heading: 'Individual Circuits' },
              {
                type: 'entities',
                show_header_toggle: false,
                state_color: true,
                entities: [
                  { entity: 'sensor.hvac_power', name: 'HVAC' },
                  { entity: 'sensor.kitchen_power', name: 'Kitchen' },
                  { entity: 'sensor.office_power', name: 'Office' }
                ]
              }
            ]
          }
        ]
      }]
    }
  });

  return { success: true };
}
```

---

## Example 5: Security Dashboard

```javascript
async () => {
  const hass = document.querySelector('home-assistant')?.hass;

  await hass.callWS({
    type: 'lovelace/dashboards/create',
    url_path: 'security-panel',
    title: 'Security',
    icon: 'mdi:shield-home',
    show_in_sidebar: true,
    require_admin: false
  });

  await hass.callWS({
    type: 'lovelace/config/save',
    url_path: 'security-panel',
    config: {
      views: [{
        title: 'Security',
        type: 'sections',
        sections: [
          {
            type: 'grid',
            cards: [
              { type: 'heading', heading: 'Alarm' },
              {
                type: 'alarm-panel',
                entity: 'alarm_control_panel.home',
                states: ['arm_home', 'arm_away', 'arm_night']
              }
            ]
          },
          {
            type: 'grid',
            cards: [
              { type: 'heading', heading: 'Doors & Windows' },
              {
                type: 'entities',
                state_color: true,
                entities: [
                  { entity: 'binary_sensor.front_door', name: 'Front Door' },
                  { entity: 'binary_sensor.back_door', name: 'Back Door' },
                  { entity: 'binary_sensor.garage_door', name: 'Garage' }
                ]
              }
            ]
          },
          {
            type: 'grid',
            cards: [
              { type: 'heading', heading: 'Motion' },
              {
                type: 'entities',
                state_color: true,
                entities: [
                  { entity: 'binary_sensor.living_room_motion', name: 'Living Room' },
                  { entity: 'binary_sensor.hallway_motion', name: 'Hallway' },
                  { entity: 'binary_sensor.garage_motion', name: 'Garage' }
                ]
              }
            ]
          },
          {
            type: 'grid',
            cards: [
              { type: 'heading', heading: 'Locks' },
              {
                type: 'tile',
                entity: 'lock.front_door',
                name: 'Front Door',
                features: [{ type: 'lock-commands' }]
              },
              {
                type: 'tile',
                entity: 'lock.back_door',
                name: 'Back Door',
                features: [{ type: 'lock-commands' }]
              }
            ]
          }
        ]
      }]
    }
  });

  return { success: true };
}
```

---

## File System Structure Reference

When dashboards are created via websocket API, Home Assistant stores them in `/config/.storage/`:

| File | Purpose |
|------|---------|
| `lovelace_dashboards` | Registry of all user-created dashboards |
| `lovelace.<dashboard_id>` | Individual dashboard configuration |
| `lovelace_resources` | Custom card resources (HACS cards, etc.) |

**Note**: Direct file editing requires HA restart. Always prefer the websocket API.
