# Home Assistant Dashboard Agent Skill

A Claude Code plugin/skill for creating and modifying Home Assistant Lovelace dashboards using the websocket API via Chrome DevTools.

## Features

- Create dashboards dynamically via Home Assistant's websocket API
- Query entities with rich metadata (areas, devices, states)
- Build dense, process-control style layouts
- Organize dashboards by area automatically
- No Home Assistant restart required

## Requirements

- [Claude Code](https://claude.ai/claude-code)
- Chrome DevTools MCP server (`mcp__chrome-devtools`)
- Access to a Home Assistant instance

## Prerequisites: Chrome DevTools MCP Setup

This skill requires the Chrome DevTools MCP server to interact with Home Assistant's web interface.

```bash
claude mcp add chrome-devtools npx chrome-devtools-mcp@latest
```

After adding the MCP server, restart Claude Code to load the new configuration.

## Installation

### Option 1: Install from GitHub

```
/plugin install hass-dashboard-agent-skill@your-marketplace
```

### Option 2: Clone and install locally

```bash
git clone https://github.com/yourname/hass_dashboard_agent_skill.git
cd hass_dashboard_agent_skill
```

Then in Claude Code:
```
/plugin install .
```

### Option 3: Add to your project

Copy the `skills/home-assistant-dashboards/` directory to your project's `.claude/skills/` folder.

## Usage

Once installed, simply ask Claude to create a Home Assistant dashboard:

- "Create a lights dashboard organized by area"
- "Make a temperature monitoring dashboard"
- "Build a dense power monitoring panel"

Claude will:
1. Ask for your Home Assistant URL
2. Connect via Chrome DevTools
3. Query your entities
4. Create the dashboard

## Skill Documentation

- [SKILL.md](skills/hass-dashboard-agent-skill/SKILL.md) - Main workflow and quick reference
- [CARDS.md](skills/hass-dashboard-agent-skill/CARDS.md) - Card types reference
- [VIEWS.md](skills/hass-dashboard-agent-skill/VIEWS.md) - View types and layouts
- [EXAMPLES.md](skills/hass-dashboard-agent-skill/EXAMPLES.md) - Complete examples
- [TROUBLESHOOTING.md](skills/hass-dashboard-agent-skill/TROUBLESHOOTING.md) - Common issues

## Example Output

Dense lights dashboard with all lights organized by area:

```
┌─────────────┬─────────────┬─────────────┬─────────────┐
│   Master    │   Office    │   Kitchen   │     Den     │
├─────────────┼─────────────┼─────────────┼─────────────┤
│ Fan  Lamp1  │ Bookcase    │ Bar Light   │ Fireplace   │
│ 100% 100%   │ BC Cool     │ On          │ Off         │
│             │ BC Warm     │             │             │
│ Lamp2       │ Baker W/C   │             │             │
│ 1%          │ Desk Camera │             │             │
├─────────────┼─────────────┼─────────────┼─────────────┤
│  Back Deck  │   Outside   │    Bonus    │  Kids Room  │
├─────────────┼─────────────┼─────────────┼─────────────┤
│ Fan DeckFan │ Porch       │ OH 1  OH 2  │ Overhead    │
│ Lamp        │ Garage      │             │             │
│             │ NE Yard     │             │             │
└─────────────┴─────────────┴─────────────┴─────────────┘
```

## License

MIT

## Contributing

Pull requests welcome! Please ensure skills follow the [Claude Code skill format](https://docs.anthropic.com/claude-code/skills).
