---
description: "godon MCP interface — connect any MCP-compatible LLM client to manage systemtenders, targets, credentials, and steerwishes, and read the live causal map, via the Model Context Protocol."
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

godon ships an MCP server (`godon-mcp`) as part of the Helm chart. Any MCP-compatible client — Claude Desktop, Claude Code, opencode, or any MCP SDK — can connect and manage systemtenders, targets, credentials, and steerwishes, and read the live causal map directly.

### Overview

| | |
|---|---|
| **Protocol** | MCP streamable HTTP (2025-03-26) at `/mcp`; legacy SSE (2024-11-05) at `/sse` |
| **Service** | `godon-mcp` in namespace `godon` |
| **Port** | 3001 |
| **Endpoint** | `http://<host>:3001/mcp` — or `http://<host>:3001/sse` for legacy SSE clients |

Management and steering tools proxy the godon API, so the systemtender, credential, target, and steerwish surfaces mirror the REST API. The connectome tools read the causal map service directly — the same map the REST `/connectome` family relays — so an LLM sees exactly what the system currently believes.

### Available Tools

#### Systemtenders

| Tool | Description |
|------|-------------|
| `systemtender_list` | List all optimization systemtenders with their current status (active, stopped, error) |
| `systemtender_get` | Get a systemtender's full configuration, status, and creation time |
| `systemtender_create` | Create and start a new optimization systemtender (name + godon v0.3 config) |
| `systemtender_stop` | Gracefully stop a running systemtender — workers finish their current trial |
| `systemtender_start` | Resume a stopped systemtender, continuing from its trial history |
| `systemtender_delete` | Delete a systemtender and all its data; `force=true` cancels running workers immediately |

#### Credentials

| Tool | Description |
|------|-------------|
| `credential_list` | List all stored credentials (SSH keys, API tokens, etc.) |
| `credential_create` | Register a credential: SSH key, API token, database connection, or HTTP basic auth |
| `credential_get` | Get details of a specific credential |
| `credential_delete` | Delete a stored credential |

#### Targets

| Tool | Description |
|------|-------------|
| `target_list` | List all registered target systems that systemtenders optimize against |
| `target_create` | Register a target system: SSH server or HTTP API |
| `target_get` | Get details of a specific target system |
| `target_delete` | Delete a registered target system |

#### Steerwishes

| Tool | Description |
|------|-------------|
| `steerwish_list` | List all declared steerwishes with their derived lifecycle state (declared, planned, refused, acted, landed, missed, re_opened, closed) |
| `steerwish_declare` | Declare a steerwish: a named measured outcome to bring into a band and hold. The map plans the input setting; a refusal names its binding constraint. Omitted budget means upkeep indefinitely; only the standing regime exists today |
| `steerwish_get` | Get one steerwish with its full event history |
| `steerwish_close` | Close a steerwish and release the hold (idempotent) |

#### The map (connectome)

| Tool | Description |
|------|-------------|
| `connectome_get` | The live map: every node and characterized edge, with fitted response, confidence, and noise floor. Partial by design and always aging — check freshness before trusting |
| `connectome_curves` | The measured response curves: per edge, probe levels with measured shifts and honest error bars |
| `connectome_artifact` | The exported map artifact: the full connectome with curves and metadata, as persisted at last build |
| `connectome_predict` | Predict the one-hop shift at a receiver for a push on a sender, linearized from the measured map. Reads never touch the system |
| `connectome_predict_multihop` | Predict the composed cascade shift along a measured path, linearized. Reads never touch the system |
| `connectome_impact` | What a given systemtender's probing has moved: its measured impact across the map |
| `connectome_causes` | What feeds a given systemtender's nodes: the measured causes upstream of its patch of the map |

#### Health

| Tool | Description |
|------|-------------|
| `health` | Check platform health |

### Connecting

Port-forward the MCP service to your local machine:

```bash
kubectl port-forward svc/godon-mcp -n godon 3001:3001
```

Then point your MCP client at `http://localhost:3001/mcp`. Clients that only speak the legacy SSE transport connect to `http://localhost:3001/sse` instead.

#### Claude Desktop

Add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "godon": {
      "url": "http://localhost:3001/mcp"
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
      "url": "http://localhost:3001/mcp"
    }
  }
}
```

#### Python MCP SDK

```python
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client

async with streamablehttp_client("http://localhost:3001/mcp") as (read, write, _):
    async with ClientSession(read, write) as session:
        await session.initialize()
        tools = await session.list_tools()
```

Legacy SSE clients use `from mcp.client.sse import sse_client` with `http://localhost:3001/sse`.

#### TypeScript MCP SDK

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StreamableHTTPClientTransport } from "@modelcontextprotocol/sdk/client/streamableHttp.js";

const transport = new StreamableHTTPClientTransport(new URL("http://localhost:3001/mcp"));
const client = new Client({ name: "my-client", version: "1.0.0" });
await client.connect(transport);
const tools = await client.listTools();
```

### Example Usage

Once connected, an LLM can create and manage optimization runs directly. For example, to create a TCP tuning systemtender:

```
Create a systemtender named "tcp-tuning" that optimizes net.ipv4.tcp_rmem
between 4096 and 6291456 targeting host 10.0.0.5, measuring RTT as the objective.
```

The LLM translates this into a `systemtender_create` tool call with the appropriate godon v0.3 configuration.

To hold an outcome in its band, declare the wish in plain terms:

```
Keep chainend.shift between -0.05 and 0.05, upkeep indefinitely, and never move
any input further than 20 percent from its neutral point.
```

The LLM translates this into a `steerwish_declare` call: an outcome from the map's outcome registry, a band, optional limits. Planning the input setting is the map's job; a refusal names its binding constraint. A standing wish is held until closed.

And to ask what the system actually believes:

```
How does rmem move RTT according to the map, and how fresh is that knowledge?
```

`connectome_get` and `connectome_curves` answer with the measured map: fitted responses with confidence and noise floor, probe levels with honest error bars. The map is partial by design — curves exist only where the system has probed — and always aging.
