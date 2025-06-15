---
title: "Convex MCP Server"
sidebar_position: 300
description: "Convex MCP server"
---

The Convex
[Model Context Protocol](https://docs.cursor.com/context/model-context-protocol)
(MCP) server provides several tools that allow AI agents to interact with your
Convex deployment.

## Setup

Add the following command to your MCP servers configuration:

`npx -y convex@latest mcp start`

See editor specific instructions:

- [Cursor](/docs/ai/using-cursor.mdx#setup-the-convex-mcp-server)
- [Windsurf](/docs/ai/using-windsurf.mdx#setup-the-convex-mcp-server)

## Available Tools

### Deployment Tools

- **`status`**: Queries available deployments and returns a deployment selector
  that can be used with other tools. This is typically the first tool you'll use
  to find your Convex deployment.

### Table Tools

- **`tables`**: Lists all tables in a deployment along with their:

  - Declared schemas (if present)
  - Inferred schemas (automatically tracked by Convex)
  - Table names and metadata

- **`data`**: Allows pagination through documents in a specified table.

- **`runOneoffQuery`**: Enables writing and executing sandboxed JavaScript
  queries against your deployment's data. These queries are read-only and cannot
  modify the database.

### Function Tools

- **`functionSpec`**: Provides metadata about all deployed functions, including:

  - Function types
  - Visibility settings
  - Interface specifications

- **`run`**: Executes deployed Convex functions with provided arguments.

### Environment Variable Tools

- **`envList`**: Lists all environment variables for a deployment
- **`envGet`**: Retrieves the value of a specific environment variable
- **`envSet`**: Sets a new environment variable or updates an existing one
- **`envRemove`**: Removes an environment variable from the deployment

[Read more about how to use the Convex MCP Server](https://stack.convex.dev/convex-mcp-server)

---

## Windsurf Editor – Model Context Protocol (MCP)

**MCP (Model Context Protocol)** is a protocol that enables LLMs to access custom tools and services. An MCP client (Cascade, in this case) can make requests to MCP servers to access tools that they provide. Cascade now natively integrates with MCP, allowing you to bring your own selection of MCP servers for Cascade to use.  
See the [official MCP docs](#) for more information.

> **Note:** Enterprise users must manually turn this on via settings.

---

### ⚙️ Configuring MCP

To set up MCP with Cascade:

1. **Navigate to:**  
   - `Windsurf → Settings → Advanced Settings`  
   - or use `Command Palette → Open Windsurf Settings Page`.
2. **Scroll down to the Cascade section**  
   - Here you’ll find options to add a new server, view existing servers, and a button to view the raw JSON config file at `mcp_config.json`.

**Windsurf supports two transport types for MCP servers:**
- `stdio`
- `/sse` (for servers with an endpoint like `https://<your-server-url>/sse`)

---

### ➕ Adding a New Server

To add a new server:

1. Press the **“Add Server”** button in the Cascade settings.
2. Choose from pre-populated servers, or click **“Add custom server +”** to add your own server directly in `mcp_config.json`.

> **Limit:**  
> The number of tools you can access from MCP servers at one time is **limited to 50**.  
> If adding a new server exceeds this limit, you must remove an existing server.

**After adding a new MCP server, press the refresh button.**

---

### 📝 `mcp_config.json`

The `~/.codeium/windsurf/mcp_config.json` file contains a list of servers that Cascade can connect to.

**Schema:**  
The JSON should follow the same schema as the config file for Claude Desktop.

#### Example configuration (Google Maps server):

```json
{
  "mcpServers": {
    "google-maps": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-google-maps"
      ],
      "env": {
        "GOOGLE_MAPS_API_KEY": "<YOUR_API_KEY>"
      }
    }
  }
}
```

> Be sure to provide the required arguments and environment variables for the servers that you want to use.

See the [official MCP server reference repository](#) or [OpenTools](#) for example servers.

---

### 📝 Notes & Limitations

- MCP tool calls can invoke code written by arbitrary server implementers.  
  **We do not assume liability for MCP tool call failures.**
- **MCP tool calls will consume credits regardless of success or failure.**
- Currently, only tools are supported (not prompts or resources).  
  Cascade can use a server’s tools, but not other endpoints that a server exposes.
- **Tools that output images are not supported.**

---

