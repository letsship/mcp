<p align="center"><img src="assets/ship.png" alt="SHIP" width="96" height="96"></p>

# SHIP MCP server

[![MCP Registry](https://img.shields.io/badge/MCP%20Registry-ai.letsship%2Fship-e84826)](https://registry.modelcontextprotocol.io/v0.1/servers/ai.letsship%2Fship/versions/latest)
[![Transport](https://img.shields.io/badge/transport-Streamable%20HTTP-0f0a07)](https://letsship.ai/docs/mcp)
[![Auth](https://img.shields.io/badge/auth-OAuth%202.1%20%7C%20token-0f0a07)](https://letsship.ai/docs/mcp#connect-a-client)
[![Docs](https://img.shields.io/badge/docs-letsship.ai-0f0a07)](https://letsship.ai/docs/mcp)

Assign Linear or GitHub issues to autonomous coding agents, and get pull requests that pass automated code review and tests. SHIP runs agentic coding in your own GitHub repository. From your assistant, file missions, follow them, send instructions and read the proof of each run.

SHIP runs an agentic SDLC as a managed AI software factory. A Planner, a Builder, a Reviewer and a Tester each do one stage, and your CI is a gate between them. Each role runs the coding agent and the model that you choose: Claude Code, Codex, OpenCode, Kilo Code or the pi coding agent.

[![Install in VS Code](https://img.shields.io/badge/VS%20Code-Install%20SHIP-0098FF)](https://vscode.dev/redirect/mcp/install?name=ship&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fmcp.letsship.ai%2Fmcp%22%7D)
[![Install in VS Code Insiders](https://img.shields.io/badge/VS%20Code%20Insiders-Install%20SHIP-24bfa5)](https://insiders.vscode.dev/redirect/mcp/install?name=ship&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fmcp.letsship.ai%2Fmcp%22%7D)
[![Add to Cursor](https://cursor.com/deeplink/mcp-install-dark.svg)](cursor://anysphere.cursor-deeplink/mcp/install?name=ship&config=eyJ1cmwiOiJodHRwczovL21jcC5sZXRzc2hpcC5haS9tY3AifQ==)

## Endpoint

```text
https://mcp.letsship.ai/mcp
```

The transport is Streamable HTTP. The server is stateless, so a client can reconnect at any time and lose nothing.

## Sign in

**OAuth 2.1.** A client that supports MCP authorization finds the sign-in on its own. A request that needs a token gets `401` with a `WWW-Authenticate` header that points to
`https://mcp.letsship.ai/.well-known/oauth-protected-resource/mcp`. SHIP supports dynamic client registration and client ID metadata documents, so you do not register a client first. A person approves the request in the SHIP console.

**Personal access token.** A client without OAuth sends `Authorization: Bearer pat_...`. Create a token at https://letsship.ai/settings/tokens, and bind it to one project so that it can reach only that project.

## Install

**Claude Code**

```bash
claude mcp add --transport http ship https://mcp.letsship.ai/mcp
```

**Claude and ChatGPT.** Add a custom connector with the URL `https://mcp.letsship.ai/mcp`, then sign in when the app asks.

**Cursor.** Use the button above, or add this to `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "ship": {
      "url": "https://mcp.letsship.ai/mcp"
    }
  }
}
```

**VS Code.** Use the buttons above, or add this to `.vscode/mcp.json`:

```json
{
  "servers": {
    "ship": {
      "type": "http",
      "url": "https://mcp.letsship.ai/mcp"
    }
  }
}
```

**Codex.** Add this to `~/.codex/config.toml`:

```toml
[mcp_servers.ship]
url = "https://mcp.letsship.ai/mcp"
```

**Grok.** Open grok.com/connectors, select **New Connector**, then **Custom**, paste `https://mcp.letsship.ai/mcp` and sign in.

**xAI API.** Pass the URL as a remote MCP tool, with a token in `headers`:

```json
{
  "type": "mcp",
  "server_url": "https://mcp.letsship.ai/mcp",
  "server_label": "ship",
  "headers": {
    "Authorization": "Bearer pat_your_token_here"
  }
}
```

**A client that speaks only stdio** connects through `mcp-remote`:

```json
{
  "mcpServers": {
    "ship": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://mcp.letsship.ai/mcp"
      ]
    }
  }
}
```

**With a token.** Add the header to any of the HTTP entries above:

```json
{
  "mcpServers": {
    "ship": {
      "url": "https://mcp.letsship.ai/mcp",
      "headers": {
        "Authorization": "Bearer pat_your_token_here"
      }
    }
  }
}
```

## Tools

Each tool declares `readOnlyHint`, `destructiveHint` and `openWorldHint`, so your client can run a tool that reads at once and ask you before one that removes or replaces something or opens a payment page. The token's scopes decide what each tool can do, and a refusal names what the token lacks.

> [!IMPORTANT]
> `start_mission`, `create_mission`, `run_scheduled_task` start a mission, and a mission start costs one credit. Messages to a running mission are free.

### Missions

| Tool | Kind | What it does |
| --- | --- | --- |
| `start_mission` | reaches outside SHIP | Give an issue that is already in your tracker to the fleet. By default, the mission starts at the Planner. With a plan, it starts at the Builder. With an open pull request, it starts at the Reviewer or the QA agent |
| `create_mission` | reaches outside SHIP | Create the issue and start the mission in one call, for work that has no issue yet |
| `send_mission_instruction` | reaches outside SHIP | Tell an active mission what to change, or record a note for the agent that runs next |
| `inspect_mission` | reads | The full record of a mission: its stages, verdicts, provenance and session logs |
| `list_missions` | reads | The missions of your projects, with what each mission does now |
| `get_mission` | reads | One mission: its stage, its review and test results, its pull request, its agent runs and the run with the proof. With `view: "full"`, also its events and the outcome of each stage |
| `pause_mission` | writes | Hold a mission before its next stage. The stage that runs now finishes |
| `resume_mission` | reaches outside SHIP | Continue a paused mission. When a checkpoint holds the mission, this approves the result of that stage |
| `stop_mission` | removes | Stop a mission and every agent run of it. The pull request stays open |
| `approve_mission_ceiling` | reaches outside SHIP | Let a mission continue after it stops at its credit limit. The mission then uses more credits |
| `merge_mission` | removes | Merge the pull request of a mission, with the merge method of the repository. On a merge conflict, SHIP starts the Builder to fix the conflict and merges later |
| `decline_mission` | removes | Close the pull request of a mission without a merge |
| `set_agents_override` | writes | Change the harness, model, prompt or checkpoint of one role for one mission. The override goes on top of `ship.yml` and does not change the file |
| `get_agents_override` | reads | Read the override that applies to a mission now |
| `clear_agents_override` | removes | Remove the override, so the mission uses the project's configuration again |

### Projects and proof

| Tool | Kind | What it does |
| --- | --- | --- |
| `list_projects` | reads | The projects that the token can reach |
| `get_project` | reads | The configuration and connections of one project |
| `list_artifacts` | reads | The proof that a run produced, such as test traces and screenshots |
| `get_artifact` | reads | One artifact, with a link to open it |

### Scheduled tasks

| Tool | Kind | What it does |
| --- | --- | --- |
| `list_scheduled_tasks` | reads | The missions that a project runs on a schedule |
| `get_scheduled_task` | reads | One scheduled task, with its schedule and prompt |
| `create_scheduled_task` | writes | Add a mission that runs on a schedule |
| `update_scheduled_task` | writes | Change the schedule or prompt of a scheduled task, or disable it |
| `run_scheduled_task` | reaches outside SHIP | Run a scheduled task now, outside its schedule |
| `list_scheduled_task_runs` | reads | What past runs did, and what they cost |
| `list_scheduled_task_templates` | reads | Templates that you can copy to start a scheduled task |

### MCP servers for your agents

| Tool | Kind | What it does |
| --- | --- | --- |
| `list_mcp_servers` | reads | The servers that the MCP file of a project declares, each with its binding and status |
| `list_mcp_bindings` | reads | Each binding in the projects of your organization, so you can see where each credential is used |
| `bind_mcp_server` | replaces | Choose the agents, transport, approved tools and credential for one server |
| `unbind_mcp_server` | removes | Remove a binding and delete its credential |
| `refresh_mcp_server_tools` | removes | Read the current tool list of the server, so that you can approve tools |
| `start_mcp_server_oauth` | reaches outside SHIP | Get a sign-in link for a remote server, for a person to open in their browser |
| `set_mcp_config_path` | writes | Set a different MCP file for a project, such as `.cursor/mcp.json` |

### Agent runs

| Tool | Kind | What it does |
| --- | --- | --- |
| `list_agent_runs` | reads | Agent runs, live and finished, for one mission, one role or one status |
| `get_agent_run` | reads | The detail and resource use of one run |
| `get_agent_run_logs` | reads | What an agent did inside a run |
| `watch_agent_run` | reads | Wait on one run and follow its events, until the run ends or its mission moves to another stage |
| `stop_agent_run` | removes | Stop the agent in one run before it finishes |

### Preview environments

| Tool | Kind | What it does |
| --- | --- | --- |
| `list_environments` | reads | Preview environments that SHIP deployed for open pull requests |
| `get_environment` | reads | One preview environment, with its URL |
| `destroy_environment` | removes | Remove a preview environment early |

### Credits and notices

| Tool | Kind | What it does |
| --- | --- | --- |
| `get_billing` | reads | The credit balance, the credit limit for each mission and the recent use. Read it before you send work |
| `list_billing_ledger` | reads | Where the credits went, with the newest entry first |
| `list_credit_offers` | reads | The credit packs and the monthly plans, with their prices |
| `start_credit_checkout` | reaches outside SHIP | Open a payment page for a credit pack, a monthly plan or more credits on a plan, and return its URL |
| `update_team_subscription` | removes | Change the monthly plan from the next renewal, or cancel it at the end of the month that you paid for |
| `request_more_credits` | reaches outside SHIP | Ask for more credits. The answer says who has the request |
| `get_notices` | reads | What needs attention in the workspace, such as a tracker that you must connect again |
| `dismiss_notice` | writes | Close a notice that you can close, such as a low balance |

### Custom sandboxes

| Tool | Kind | What it does |
| --- | --- | --- |
| `list_sandbox_templates` | reads | The sandboxes of your organization, with the active version and the latest build of each |
| `list_sandbox_starters` | reads | Sandbox definitions for Python, Java, Go, Rust and Ruby that you can start from |
| `save_sandbox_template` | writes | Create a sandbox or change one. A change makes a new version and starts its build |
| `list_sandbox_template_versions` | reads | Each saved version of a sandbox, with the active version marked |
| `activate_sandbox_template_version` | writes | Make a version the one that missions use from their next run |
| `build_sandbox_template` | writes | Build the active version now |
| `get_sandbox_build_log` | reads | One page of a build log |
| `list_sandbox_registries` | reads | The private package registries that sandboxes install from. A secret is never returned |
| `save_sandbox_registry` | replaces | Store or replace the credential for a registry. The credential never enters a sandbox |
| `get_sandbox_capacity` | reads | How many agents your missions run at once, against the limit of your organization |

### Platform operators

| Tool | Kind | What it does |
| --- | --- | --- |
| `list_orgs` | reads | Each organization on the platform. Only a SHIP operator token can call it |

### Identity

| Tool | Kind | What it does |
| --- | --- | --- |
| `whoami` | reads | The principal behind the token and the scopes of the token |
| `healthz` | reads | The status, environment and version of the API |

## Example: file a fix and answer the review

1. `list_projects` gives the project id.
2. `create_mission` files the issue and starts the mission. Write the acceptance criteria as checkboxes, because the QA agent tests the work against them.
3. The mission plans, builds, opens a pull request, waits for your CI, reviews the change, deploys a preview and tests it. This takes tens of minutes, and your assistant does not have to wait.
4. `inspect_mission` shows the stage and the verdicts. To disagree with a review, call `send_mission_instruction` with `kind: "follow_up"`. The Builder goes back to the pull request with your instruction.

Do not call `start_mission` again to send feedback. A restart carries no message, so use `send_mission_instruction`.

## Troubleshooting

| What you see | What to do |
| --- | --- |
| `401` and no sign-in window | Your client has no MCP OAuth. Send a personal access token in the `Authorization` header. |
| A tool refuses with a scope name | The token lacks that scope. Create a token with it, or ask an owner for the role. |
| `create_mission` refuses with `credits_exhausted` | The workspace has no credits left. `get_billing` shows the balance. |
| The tool list is empty, or each tool call gets `401` | The client did not finish sign-in. Sign in again from the MCP menu of your client, or remove the server and add it again. |
| `whoami` names another project | The token is bound to that project. Use a token for the project you want. |

## Skills for your coding agent

The SHIP skills teach a coding agent to hand work to SHIP from the terminal. See [letsship/skills](https://github.com/letsship/skills), or run `npx skills add https://letsship.ai`.

## Learn more

- [What an agentic SDLC is, from issue to verified pull request](https://letsship.ai/blog/agentic-sdlc)
- [What an AI software factory is, and how to build one](https://letsship.ai/blog/what-is-an-ai-software-factory)
- [SWE-in-a-team, a coding agent benchmark of 260 missions](https://letsship.ai/blog/swe-in-a-team)
- [Run the pi coding agent headless](https://letsship.ai/blog/run-pi-coding-agent-headless)
- [Send a pull request that your coding agent wrote for review and test](https://letsship.ai/platform/agent-to-pr)

## Links

- Docs: https://letsship.ai/docs/mcp
- Registry entry: `ai.letsship/ship` on https://registry.modelcontextprotocol.io
- A2A agent: https://letsship.ai/docs/a2a
- Privacy: https://letsship.ai/privacy - Terms: https://letsship.ai/terms
- Support: https://letsship.ai/support
- Security reports: support [at] letsship.ai

## License

The files in this repository are under the MIT license. The SHIP service itself is under the [SHIP terms](https://letsship.ai/terms).
