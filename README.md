# Todoist MCP Server (Zenzei Fork)

A fork of [@greirson/mcp-todoist](https://github.com/greirson/mcp-todoist) — an MCP server that connects Claude with Todoist for complete task and project management through natural language.

This fork includes bug fixes and improvements not yet in upstream. See [CHANGELOG.md](CHANGELOG.md) for details.

## Installation

### From Source (Recommended for This Fork)

```bash
# Clone the fork
git clone https://github.com/zenzeizen/mcp-todoist.git

# Install and build
cd mcp-todoist
npm install
npm run build
```

### Configure Claude Code

Add to your global `~/.claude.json` under `mcpServers`:

```json
{
  "todoist": {
    "type": "stdio",
    "command": "node",
    "args": ["/path/to/mcp-todoist/dist/index.js"],
    "env": {
      "TODOIST_API_TOKEN": "your_api_token_here"
    }
  }
}
```

### Configure Claude Desktop

Add to `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows):

```json
{
  "mcpServers": {
    "todoist": {
      "command": "node",
      "args": ["/path/to/mcp-todoist/dist/index.js"],
      "env": {
        "TODOIST_API_TOKEN": "your_api_token_here"
      }
    }
  }
}
```

Get your API token from [Todoist Settings > Integrations > Developer](https://todoist.com/app/settings/integrations/developer).

## Fork Changes

Changes in this fork on top of upstream v1.0.3:

- **Fix HTML-escaping of API payloads** — the sanitizer was encoding `/`, `&`, `"`, `'` as HTML entities in content sent to the REST API, mangling URLs and causing 400 errors on task creation with description + due_string
- **Fix stateful regex bypass** — `MALICIOUS_PATTERNS` used `/g` flag, causing `lastIndex` to persist across `.test()` calls and alternately bypassing validation
- **Fix missing update sanitization** — `handleUpdateTask` and `handleBulkUpdateTasks` passed content/description to the API without validation
- **Remove false-positive XSS patterns** — patterns like `on\w+\s*=` and `file:` silently stripped legitimate text from task descriptions
- **Add `section_id` filter to `todoist_task_get`** — filter tasks by section, supported by the Todoist API but not previously exposed
- **Add `sectionId` to task display output** — visible in `formatTaskForDisplay` results
- **Wire `assignee_id` through create/update** — existed in types but was never passed to the API
- **Conditional payload construction** — only include `description`, `dueString` in API payloads when actually provided
- **Clarify labels on update** — tool description now warns that labels replace the full set, not append

## Features

- **19 MCP Tools** for complete Todoist management
- **Task Management**: Create, update, delete, complete, reopen tasks with priorities, due dates, labels, assignees
- **Bulk Operations**: Process multiple tasks efficiently
- **Subtasks**: Hierarchical task management with completion tracking
- **Projects & Sections**: Full organization support with section-based filtering
- **Labels, Filters, Reminders**: Pro/Business features supported
- **Natural Language**: Quick add with Todoist's natural language parsing
- **Dry-Run Mode**: Test operations without making changes

## Dry-Run Mode

Test operations without making real changes. Add `DRYRUN=true` to your env config:

```json
{
  "todoist": {
    "command": "node",
    "args": ["/path/to/mcp-todoist/dist/index.js"],
    "env": {
      "TODOIST_API_TOKEN": "your_api_token_here",
      "DRYRUN": "true"
    }
  }
}
```

- **Validates** operations against real API data
- **Simulates** mutations (create, update, delete, complete)
- **Passes through** read operations unchanged
- **Logs** with `[DRY-RUN]` prefixes

## Tools Overview

**19 MCP tools** for complete Todoist management:

| Tool                    | Actions                                                                 | Description                     |
| ----------------------- | ----------------------------------------------------------------------- | ------------------------------- |
| `todoist_task`          | create, get, update, delete, complete, reopen, quick_add                | Complete task management        |
| `todoist_task_bulk`     | bulk_create, bulk_update, bulk_delete, bulk_complete                    | Efficient multi-task operations |
| `todoist_subtask`       | create, bulk_create, convert, promote, hierarchy                        | Hierarchical task management    |
| `todoist_project`       | create, get, update, delete, archive, collaborators                     | Project CRUD and sharing        |
| `todoist_project_ops`   | reorder, move_to_parent, get_archived                                   | Advanced project operations     |
| `todoist_section`       | create, get, update, delete, move, reorder, archive                     | Section management              |
| `todoist_label`         | create, get, update, delete, stats                                      | Label management with analytics |
| `todoist_comment`       | create, get, update, delete                                             | Task/project comments           |
| `todoist_reminder`      | create, get, update, delete                                             | Reminder management (Pro)       |
| `todoist_filter`        | create, get, update, delete                                             | Custom filters (Pro)            |
| `todoist_collaboration` | invitations, notifications, workspace operations                        | Team collaboration features     |
| `todoist_user`          | info, productivity_stats, karma_history                                 | User profile and stats          |
| `todoist_utility`       | test_connection, test_features, test_performance, find/merge duplicates | Testing and utilities           |
| `todoist_activity`      | get_log, get_events, get_summary                                        | Activity audit trail            |
| `todoist_task_ops`      | move, reorder, close                                                    | Advanced task operations        |
| `todoist_completed`     | get, get_all, get_stats                                                 | Completed task retrieval        |
| `todoist_backup`        | list, download                                                          | Automatic backup access         |
| `todoist_notes`         | create, get, update, delete                                             | Project notes (collaborators)   |
| `todoist_shared_labels` | create, get, rename, remove                                             | Workspace labels (Business)     |

For detailed tool documentation with parameters and examples, see **[TOOLS_REFERENCE.md](TOOLS_REFERENCE.md)**.

## Usage Notes

- **Labels on update replace the full set** — always pass all labels you want to keep, not just new ones
- **Task assignment** requires a shared project — use the `collaborators` action to find user IDs
- **Section filtering** — use `section_id` with `todoist_task_get` to scope queries to a specific section
- **Duration** requires `due_string` with a time component (e.g., "tomorrow at 2pm")

## Development

```bash
npm run build          # Compile TypeScript
npm run watch          # Watch mode
npm test               # Run tests
npm run test:coverage  # Tests with coverage
npm run lint           # Lint
npm run lint:fix       # Auto-fix lint issues
npm run format         # Format with Prettier
```

## Upstream

Based on [greirson/mcp-todoist](https://github.com/greirson/mcp-todoist) v1.0.3. MIT licensed.

## Troubleshooting

**Connection errors:** Verify your API token at [Todoist Settings](https://todoist.com/app/settings/integrations/developer).

**MCP not loading:** Confirm the path in your config points to `dist/index.js` and the build is current (`npm run build`).

**After pulling updates:** Always run `npm install && npm run build` to rebuild.
