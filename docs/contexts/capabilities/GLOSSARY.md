# Capabilities

Model-facing capability plugins: each capability is one seam, the providers that implement it, and the tool that exposes it to the model. This context also owns delegation, orchestration, background work, and runtime self-modification.

Members: `packages/web` (5), `packages/mcp` (3), `packages/skill` (5), `packages/subagent` (9), `packages/team` (2), `packages/workflow` (3), `packages/jobs` (3), `packages/extensions` (4), `packages/browser-use` (1), `packages/computer-use` (1) — 36 packages. `experimental/` providers are filed here by subject, not as a context of their own.

*Seam, provider, and service definition are defined once in the [root glossary](../../glossary.md#capability-seam); this page does not redefine them. The model-visible names behind these terms are listed in the [tool catalog](../../tool-catalog.md).*

## Language

### Web access

**Provider availability**: The cheap local check — credential or configuration presence — that makes a web provider eligible for execution-time selection. It is not a health signal and must not touch the network. _Lives in_: [`packages/web`](../../../packages/web/README.md), [web access](../../subsystems/web.md). _Avoid_: health, status.

**Readiness**: A configuration-only check of whether the web provider an operation would select can authenticate that operation for the current model route. The search and fetch tools are activated only when it answers true. _Lives in_: [`packages/web`](../../../packages/web/README.md), [web access](../../subsystems/web.md).

### MCP

**MCP server**: An external Model Context Protocol endpoint configured in a caller's scope. It contributes ordinary harness tools, resources, and instructions, and is visible only to callers that can see it. _Lives in_: [`packages/mcp`](../../../packages/mcp/README.md), [MCP](../../subsystems/mcp.md).

**Server instructions**: Guidance an MCP server publishes into the system prompt as a scoped, server-attributed section; blank instructions add no section. _Lives in_: [`packages/mcp/mcp-client`](../../../packages/mcp/mcp-client/README.md), [MCP](../../subsystems/mcp.md).

### Skills

**Skill**: Reusable task instructions an agent loads on demand by name. A skill is model-invocable, user-invocable, or both, and its body is read only when it is loaded. _Lives in_: [`packages/skill`](../../../packages/skill/README.md), [skills](../../subsystems/skills.md). _Avoid_: prompt, playbook, snippet.

**Skill catalog**: The durable, session-visible list of skill names and descriptions the model is offered, replaced as discovery changes. It carries no bodies, paths, or providers. _Lives in_: [`packages/skill/tool-skill`](../../../packages/skill/tool-skill/README.md), [skills](../../subsystems/skills.md). _Avoid_: skill list, index.

### Delegation

**Subagent**: A child agent that a parent creates to carry out a delegated task. A local child has a durable Session and at most one live activation. _Lives in_: [`packages/subagent`](../../../packages/subagent/README.md), [subagents](../../subsystems/subagent.md). _Avoid_: child agent, worker.

**Continuable child**: A local subagent that accepts follow-up turns after its first activation and can be cold-resumed from its own Session later. _Lives in_: [`packages/subagent`](../../../packages/subagent/README.md), [subagents](../../subsystems/subagent.md).

**Agent name**: The model-facing reference to a member of a subagent tree: `lead` for the root, otherwise a slash-separated path from the root when every link is named. Session ids remain the only execution identity. _Lives in_: [`packages/subagent`](../../../packages/subagent/README.md), [subagents](../../subsystems/subagent.md). _Avoid_: agent id.

### Shared task board

**Team mode**: The root Session's saved selection that enables the shared task board and its tools for an entire subagent tree. Descendants share the root's choice, and only the root can change it. _Lives in_: [`packages/team`](../../../packages/team/README.md), [agent teams](../../subsystems/agent-team.md).

**Team task**: A shared board item with a status, one owner, blocker edges, and advisory write scopes. Readiness is derived from those edges and is never stored on the task. _Lives in_: [`packages/team`](../../../packages/team/README.md), [agent teams](../../subsystems/agent-team.md). _Avoid_: todo, ticket.

### Orchestration

**Workflow run**: One execution of a model-written orchestration script that fans work out to subagents and settles with a single JSON value. _Lives in_: [`packages/workflow`](../../../packages/workflow/README.md), [workflow](../../subsystems/workflow.md). _Avoid_: script, pipeline.

### Background work

**Background job**: A long-running execution registered with the job runtime and owned by the Session that started it, so the model can read, wait on, and cancel it without blocking its turn. _Lives in_: [`packages/jobs`](../../../packages/jobs/README.md), [jobs](../../subsystems/jobs.md). _Avoid_: task, process.

**Output ring**: The bounded per-job output buffer addressed by absolute byte offsets. The model reads it through a consuming cursor while any number of observers read without consuming. _Lives in_: [`packages/jobs`](../../../packages/jobs/README.md), [jobs](../../subsystems/jobs.md). _Avoid_: log, buffer.

### Runtime self-modification

**Dynamic plugin**: A model-defined Cordis plugin with a host half and a browser half that is defined and run at runtime. The plugin holds the identity; each version is an immutable package. _Lives in_: [`packages/extensions`](../../../packages/extensions/README.md), [extensions](../../subsystems/extensions.md). _Avoid_: extension, add-on.

**Runtime inspection**: Read-only discovery and querying of approved host and client runtime APIs before a plugin is written. It answers metadata questions and cannot change the runtime. _Lives in_: [`packages/extensions/tool-cordis`](../../../packages/extensions/tool-cordis/README.md), [extensions](../../subsystems/extensions.md). _Avoid_: debugger, introspection.

### Device use

**Browser use**: The capability that lets a model inspect and operate web pages through one configured provider, which keeps the browser state for a live Session across turns. _Lives in_: [`packages/browser-use`](../../../packages/browser-use/README.md), [browser use](../../subsystems/browser-use.md). _Avoid_: web access, web browsing.

**Computer use**: The capability that lets a model observe and operate a local desktop through one configured provider. Cua Driver names the upstream implementation. _Lives in_: [`packages/computer-use`](../../../packages/computer-use/README.md), [computer use](../../subsystems/computer-use.md). _Avoid_: desktop control.

### Borrowed from the root glossary

**Ralph loop**: Already defined at the [root glossary](../../glossary.md#ralph); this page adds only its membership here — an experimental tool composition over the workflow engine and the subagent seam, not a capability of its own. Its round and handoff vocabulary stays at the root as well. _Lives in_: [root glossary §Ralph](../../glossary.md#ralph), [`packages/experimental/tool-ralph`](../../../packages/experimental/tool-ralph/README.md). _Avoid_: goal loop, retry loop.
