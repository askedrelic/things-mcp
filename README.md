# Things MCP Server

This [Model Context Protocol (MCP)](https://modelcontextprotocol.io/introduction) server lets you use Claude Desktop to interact with your task management data in [Things app](https://culturedcode.com/things). You can ask Claude to create tasks, analyze projects, help manage priorities, and more.

This server leverages the [Things.py](https://github.com/thingsapi/things.py) library and the [Things URL Scheme](https://culturedcode.com/things/help/url-scheme/).

<a href="https://glama.ai/mcp/servers/t9cgixg2ah"><img width="380" height="200" src="https://glama.ai/mcp/servers/t9cgixg2ah/badge" alt="Things Server MCP server" /></a>

## Features

- Access to all major Things lists (Inbox, Today, Upcoming, etc.)
- Project and area management
- Tag operations
- Advanced search capabilities
- Recent items tracking
- Detailed item information including checklists
- Support for nested data (projects within areas, todos within projects)

## Installation (for Claude Desktop)

1. Prerequisites
* Python 3.12+
* Claude Desktop
* Things 3 ("Enable Things URLs" must be turned on in Settings -> General)

2. Install uv if you haven't already:
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```
Restart your terminal afterwards.

3. Install this repo via uv tool:
```bash
uv tool install git+https://github.com/askedrelic/things-mcp
```

4. Add the Things server, to run via uvx (uv tool wrapper), to the Claude
   Desktop configuration file. This is a convenience function using jq you can copy/paste.

```bash
claude_config_update() {
   local server_name="$1"
   local server_config="$2"
   local config_file="$HOME/Library/Application Support/Claude/claude_desktop_config.json"

   [[ ! -f "$config_file" ]] && echo '{"mcpServers": {}}' > "$config_file"

   jq --arg name "$server_name" --argjson config "$server_config" \
      '.mcpServers = (.mcpServers // {}) | .mcpServers[$name] = $config' \
      "$config_file" > /tmp/claude_config.$$ && \
   mv /tmp/claude_config.$$ "$config_file"

   jq '.mcpServers' "$config_file"
}

claude_config_update things '{ "command": "uvx", "args": ["things-mcp"] }'
```
Restart the Claude Desktop app.

### Sample Usage with Claude Desktop

* "What's on my todo list today?"
* "Create a todo to pack for my beach vacation next week, include a packling checklist."
* "Evaluate my current todos using the Eisenhower matrix."
* "Help me conduct a GTD-style weekly review using Things."

#### Tips

* Create a project in Claude with custom instructions that explains how you use Things and organize areas, projects, tags, etc. Tell Claude what information you want included when it creates a new task (eg asking it to include relevant details in the task description might be helpful).
* Try adding another MCP server that gives Claude access to your calendar. This will let you ask Claude to block time on your calendar for specific tasks, create todos from upcoming calendar events (eg prep for a meeting), etc.


### Available Tools

#### List Views
- `get-inbox` - Get todos from Inbox
- `get-today` - Get todos due today
- `get-upcoming` - Get upcoming todos
- `get-anytime` - Get todos from Anytime list
- `get-someday` - Get todos from Someday list
- `get-logbook` - Get completed todos
- `get-trash` - Get trashed todos

#### Basic Operations
- `get-todos` - Get todos, optionally filtered by project
- `get-projects` - Get all projects
- `get-areas` - Get all areas

#### Tag Operations
- `get-tags` - Get all tags
- `get-tagged-items` - Get items with a specific tag

#### Search Operations
- `search-todos` - Simple search by title/notes
- `search-advanced` - Advanced search with multiple filters

#### Time-based Operations
- `get-recent` - Get recently created items

## Tool Parameters

### get-todos
- `project_uuid` (optional) - Filter todos by project
- `include_items` (optional, default: true) - Include checklist items

### get-projects / get-areas / get-tags
- `include_items` (optional, default: false) - Include contained items

### search-advanced
- `status` - Filter by status (incomplete/completed/canceled)
- `start_date` - Filter by start date (YYYY-MM-DD)
- `deadline` - Filter by deadline (YYYY-MM-DD)
- `tag` - Filter by tag
- `area` - Filter by area UUID
- `type` - Filter by item type (to-do/project/heading)

### get-recent
- `period` - Time period (e.g., '3d', '1w', '2m', '1y')


## Troubleshooting

The server includes error handling for:
- Invalid UUIDs
- Missing required parameters
- Things database access errors
- Data formatting errors

All errors are logged and returned with descriptive messages. To review the MCP logs from Claude Desktop, run this in the Terminal:
```bash
# Follow logs in real-time
tail -n 20 -f ~/Library/Logs/Claude/mcp*.log
```
