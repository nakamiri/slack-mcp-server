# slack-mcp-server

## About This Fork (by @nakamiri)

This repository is a fork of the original project:

- Upstream: https://github.com/ubie-oss/slack-mcp-server
- License: Apache-2.0 (the upstream license and notices are preserved)

### What Changed in This Fork

- Token model: Unified to User token only
  - Removed Bot token usage and the `SLACK_BOT_TOKEN` requirement
  - All Slack Web API calls now use `SLACK_USER_TOKEN`
  - `search.messages` and all other endpoints run under the User token
- Examples and env samples
  - `.env.sample`: kept only `EXMAPLES_SLACK_USER_TOKEN`
  - `examples/get_users.ts`,`examples/get_users_http.ts`: use only `EXMAPLES_SLACK_USER_TOKEN`
- Documentation updates
  - README/CLAUDE.md updated to reflect User-token-only operation
  - Added setup notes for GitHub Packages
- Package and publishing
  - Package name changed to `@nakamiri/slack-mcp-server`
  - Repository links updated to this fork
  - Added `publishConfig.registry` for GitHub Packages
  - Added `.npmrc.example` for one-shot local `npx` usage

### Notes on Behavior

- Required scopes depend on features you use (e.g., `users:read`, `conversations:read`, `search:read`, and for posting `chat:write`)
- Listing/searching visibility follows the permissions of the User token
- Breaking change: `SLACK_BOT_TOKEN` is no longer read; clients must provide `SLACK_USER_TOKEN` only


A [MCP(Model Context Protocol)](https://www.anthropic.com/news/model-context-protocol) server for accessing Slack API. This server allows AI assistants to interact with the Slack API through a standardized interface.

## Transport Support

This server supports both traditional and modern MCP transport methods:

- **Stdio Transport** (default): Process-based communication for local integration
- **Streamable HTTP Transport**: HTTP-based communication for web applications and remote clients

## Features

Available tools:

- `slack_list_channels` - List public channels in the workspace with pagination
- `slack_post_message` - Post a new message to a Slack channel
- `slack_reply_to_thread` - Reply to a specific message thread in Slack
- `slack_add_reaction` - Add a reaction emoji to a message
- `slack_get_channel_history` - Get recent messages from a channel
- `slack_get_thread_replies` - Get all replies in a message thread
- `slack_get_users` - Retrieve basic profile information of all users in the workspace
- `slack_get_user_profiles` - Get multiple users' profile information in bulk (efficient for batch operations)
- `slack_search_messages` - Search for messages in the workspace with powerful filters:
  - Basic query search
  - Location filters: `in_channel`
  - User filters: `from_user`, `with`
  - Date filters: `before` (YYYY-MM-DD), `after` (YYYY-MM-DD), `on` (YYYY-MM-DD), `during` (e.g., "July", "2023")
  - Content filters: `has` (emoji reactions), `is` (saved/thread)
  - Sorting options by relevance score or timestamp

## Quick Start

### Installation

```bash
npm install @nakamiri/slack-mcp-server
```

NOTE: This package is hosted in GitHub Packages (npm.pkg.github.com). You need a GitHub PAT with at least `read:packages` to install/run via `npx`.

### Configuration

You need to set the following environment variables:

- `SLACK_USER_TOKEN`: Slack User OAuth Token (all features use this token)
- `SLACK_SAFE_SEARCH` (optional): When set to `true`, automatically excludes private channels, DMs, and group DMs from search results. This is enforced server-side and cannot be overridden by clients.

You can also create a `.env` file to set these environment variables:

```
SLACK_USER_TOKEN=xoxp-your-user-token
SLACK_SAFE_SEARCH=true  # Optional: Enable safe search mode
```

### Usage

#### Start the MCP server

**Stdio Transport (default)**:
```bash
# One‑time setup (recommended)
npm config set @nakamiri:registry https://npm.pkg.github.com
npm config set //npm.pkg.github.com/:_authToken "<your-github-pat>"

# Run via npx
npx @nakamiri/slack-mcp-server
```

**Streamable HTTP Transport**:
```bash
npx @nakamiri/slack-mcp-server -port 3000
```

You can also run the installed module with node:
```bash
# Stdio transport
node node_modules/.bin/slack-mcp-server

# HTTP transport  
node node_modules/.bin/slack-mcp-server -port 3000
```

**Command Line Options**:
- `-port <number>`: Start with Streamable HTTP transport on specified port
- `-h, --help`: Show help message

#### Client Configuration

**For Stdio Transport (Claude Desktop, etc.)**:

```json
{
  "slack": {
    "command": "npx",
    "args": [
      "-y",
      "@nakamiri/slack-mcp-server"
    ],
    "env": {
      "SLACK_USER_TOKEN": "<your-user-token>",
      "SLACK_SAFE_SEARCH": "true"
    }
  }
}
```

**For Streamable HTTP Transport (Web applications)**:

Start the server:
```bash
SLACK_USER_TOKEN=<your-user-token> npx @nakamiri/slack-mcp-server -port 3000
```

### One‑shot (no global npm config)

Create a temporary `.npmrc` in your current directory (or use the provided `.npmrc.example`):

```bash
export GITHUB_PAT=<your-github-pat-with-read:packages>
cp .npmrc.example .npmrc
npx -y @nakamiri/slack-mcp-server
```

Connect to: `http://localhost:3000/mcp`

See [examples/README.md](examples/README.md) for detailed client examples.

#### Local Client Configuration (from repository)

If you don't want to install/publish the package and prefer running directly from your local clone, point your MCP client (e.g., Claude Desktop) to the local files.

Claude Desktop config locations:
- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Linux: `~/.config/Claude/claude_desktop_config.json`
- Windows: `C:\\Users\\<YOU>\\AppData\\Roaming\\Claude\\claude_desktop_config.json`

Example A — Use built binary (macOS/Linux):
```json
{
  "mcpServers": {
    "slack": {
      "command": "/absolute/path/to/repo/slack-mcp-server/dist/index.js",
      "args": [],
      "env": {
        "SLACK_USER_TOKEN": "xoxp-xxxxxxxx",
        "SLACK_SAFE_SEARCH": "true"
      }
    }
  }
}
```

Example B — Use Node to run dist (portable; recommended for Windows):
```json
{
  "mcpServers": {
    "slack": {
      "command": "node",
      "args": [
        "/absolute/path/to/repo/slack-mcp-server/dist/index.js"
      ],
      "env": {
        "SLACK_USER_TOKEN": "xoxp-xxxxxxxx",
        "SLACK_SAFE_SEARCH": "true"
      }
    }
  }
}
```

Example C — Run TypeScript directly (development):
```json
{
  "mcpServers": {
    "slack": {
      "command": "node",
      "args": [
        "--import",
        "/absolute/path/to/repo/slack-mcp-server/ts-node-loader.js",
        "/absolute/path/to/repo/slack-mcp-server/src/index.ts"
      ],
      "env": {
        "SLACK_USER_TOKEN": "xoxp-xxxxxxxx",
        "SLACK_SAFE_SEARCH": "true"
      }
    }
  }
}
```

Example D — Your local path on this machine (Claude Desktop, macOS):

Prereq: build once in your repo directory

```bash
cd /Users/nack/work/git-works/nakamiri/slack-mcp-server
npm ci && npm run build
```

Then configure Claude Desktop to point to the built binary:

```json
{
  "mcpServers": {
    "slack": {
      "command": "/Users/nack/work/git-works/nakamiri/slack-mcp-server/dist/index.js",
      "args": [],
      "env": {
        "SLACK_USER_TOKEN": "xoxp-xxxxxxxx",
        "SLACK_SAFE_SEARCH": "true"
      }
    }
  }
}
```

Or use Node to run the built file (portable):

```json
{
  "mcpServers": {
    "slack": {
      "command": "node",
      "args": [
        "/Users/nack/work/git-works/nakamiri/slack-mcp-server/dist/index.js"
      ],
      "env": {
        "SLACK_USER_TOKEN": "xoxp-xxxxxxxx",
        "SLACK_SAFE_SEARCH": "true"
      }
    }
  }
}
```

Notes:
- `.env` may not be loaded when launched by a client; specify `env` in the client config.
- Most desktop clients support Stdio. HTTP mode requires a client that supports Streamable HTTP.
- HTTP alternative: start server locally and connect via URL
  1) `SLACK_USER_TOKEN=... node dist/index.js -port 3000`
  2) Connect to `http://localhost:3000/mcp` from your HTTP-capable client

## Implementation Pattern

This server adopts the following implementation pattern:

1. Define request/response using Zod schemas
   - Request schema: Define input parameters
   - Response schema: Define responses limited to necessary fields

2. Implementation flow:
   - Validate request with Zod schema
   - Call Slack WebAPI
   - Parse response with Zod schema to limit to necessary fields
   - Return as JSON

For example, the `slack_list_channels` implementation parses the request with `ListChannelsRequestSchema`, calls `slackClient.conversations.list`, and returns the response parsed with `ListChannelsResponseSchema`.

## Development

### Available Scripts

- `npm run dev` - Start the server in development mode with hot reloading
- `npm run build` - Build the project for production
- `npm run start` - Start the production server
- `npm run lint` - Run linting checks (ESLint and Prettier)
- `npm run fix` - Automatically fix linting issues

### Contributing

1. Fork the repository
2. Create your feature branch
3. Run tests and linting: `npm run lint`
4. Commit your changes
5. Push to the branch
6. Create a Pull Request
