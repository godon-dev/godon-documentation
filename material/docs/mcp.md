---
description: "godon MCP interface — connect any MCP-compatible LLM client to manage breeders, credentials, and platform health via the Model Context Protocol."
---

<!--
Copyright (c) 2019 Matthias Tafelmeier.

This file is part of godon

godon is free software: you can redistribute it and/or modify
it under the terms of the GNU Affero General Public License as
published by the Free Software Foundation, either version 3 of the
License, or (at your option) any later version.

godon is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU Affero General Public License for more details.

You should have received a copy of the GNU Affero General Public License
along with this godon. If not, see <http://www.gnu.org/licenses/>.
-->

## MCP Interface

godon ships an MCP server (`godon-mcp`) as part of the Helm chart. Any MCP-compatible client — Claude Desktop, Claude Code, opencode, or any MCP SDK — can connect and manage breeders, credentials, and platform health directly.

### Overview

| | |
|---|---|
| **Protocol** | MCP 2024-11-05 over SSE |
| **Service** | `godon-mcp` in namespace `godon` |
| **Port** | 3001 |
| **Endpoint** | `http://<host>:3001/sse` |

The MCP server proxies to the godon API, so any action available through the REST API is also available as an MCP tool.

### Available Tools

| Tool | Description |
|------|-------------|
| `breeder_list` | List all optimization breeders with their current status |
| `breeder_get` | Get detailed information about a specific breeder |
| `breeder_create` | Create and start a new optimization breeder |
| `breeder_start` | Resume a previously stopped breeder |
| `breeder_stop` | Gracefully stop a running breeder |
| `breeder_delete` | Delete a breeder and all its data |
| `credential_list` | List all stored credentials |
| `credential_create` | Register a new credential (SSH key, API token, etc.) |
| `credential_get` | Get details of a specific credential |
| `credential_delete` | Delete a stored credential |
| `health` | Check platform health |

### Connecting

Port-forward the MCP service to your local machine:

```bash
kubectl port-forward svc/godon-mcp -n godon 3001:3001
```

Then point your MCP client at `http://localhost:3001/sse`.

#### Claude Desktop

Add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "godon": {
      "url": "http://localhost:3001/sse"
    }
  }
}
```

#### Claude Code / opencode

Add to your MCP server settings:

```json
{
  "mcpServers": {
    "godon": {
      "url": "http://localhost:3001/sse"
    }
  }
}
```

#### Python MCP SDK

```python
from mcp import ClientSession
from mcp.client.sse import sse_client

async with sse_client("http://localhost:3001/sse") as (read, write):
    async with ClientSession(read, write) as session:
        await session.initialize()
        tools = await session.list_tools()
```

#### TypeScript MCP SDK

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { SSEClientTransport } from "@modelcontextprotocol/sdk/client/sse.js";

const transport = new SSEClientTransport(new URL("http://localhost:3001/sse"));
const client = new Client({ name: "my-client", version: "1.0.0" });
await client.connect(transport);
const tools = await client.listTools();
```

### Example Usage

Once connected, an LLM can create and manage optimization runs directly. For example, to create a TCP tuning breeder:

```
Create a breeder named "tcp-tuning" that optimizes net.ipv4.tcp_rmem
between 4096 and 6291456 targeting host 10.0.0.5, measuring RTT as the objective.
```

The LLM translates this into a `breeder_create` tool call with the appropriate godon v0.3 configuration.
