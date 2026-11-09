# Interaction

Human decisions and session-scoped policy state that steer a running agent: approvals, questions, commands, permissions, plan, goal, schedule, and todo. The context exists because the plane where a person or an automation bridge changes what the agent may do — or whether it continues at all — carries its own vocabulary, separate from the turn machinery it steers.

## Language

### Approvals and questions

**approval request**: A same-process question about one action the agent has already proposed, naming the requesting agent and the action but deliberately not restating the action's arguments. It asks whether that exact action may proceed, never whether a category of action is permitted. _Lives in_: [packages/interaction/user-approval](../../../packages/interaction/user-approval/README.md); [User Approval](../../subsystems/approval.md). _Avoid_: permission prompt, tool confirmation.

**approval outcome**: The closed answer to an approval request: a one-shot grant, an explicit rejection, a withdrawal, or an unavailable answerer. Only the one-shot grant permits the action; every other answer, including silence from an absent answerer, fails closed. _Lives in_: [packages/interaction/user-approval](../../../packages/interaction/user-approval/README.md); [User Approval](../../subsystems/approval.md). _Avoid_: approval result, verdict.

**approval policy**: A session's standing decision about approval questions — either delegate every question to the composed answerers, or refuse every one deterministically without asking anyone. It is session policy rather than a per-action judgement, and it is one of the two knobs a permission preset switches. _Lives in_: [packages/interaction/user-approval](../../../packages/interaction/user-approval/README.md); [User Approval](../../subsystems/approval.md). _Avoid_: approval mode, safety level.

**answerer**: A composed surface that may settle a human-facing request on a session's behalf — an interactive client, or an automation bridge answering for its own agents. Answerers are consulted in order and the first claim ends the request. _Lives in_: [user-approval](../../../packages/interaction/user-approval/README.md) and [user-questions](../../../packages/interaction/user-questions/README.md); [User Approval](../../subsystems/approval.md), [User Interaction](../../subsystems/user-questions.md). _Avoid_: handler, responder, provider.

**question request**: A structured request for human input: one or more questions, each with a stable identity, optional choices, and optional supporting detail, answered as a single batch keyed by question identity. It is the agent pausing for information or a decision, as distinct from an approval request about an action. _Lives in_: [packages/interaction/user-questions](../../../packages/interaction/user-questions/README.md); [User Interaction](../../subsystems/user-questions.md). _Avoid_: ask, prompt, clarification.

**presentation intent**: A caller-declared label saying a question is a particular kind of decision — a submitted plan up for review, or a blocking permission choice — so a capable UI may present it that way. It changes presentation only; the answer encoding is identical either way. _Lives in_: [packages/interaction/user-questions](../../../packages/interaction/user-questions/README.md); [User Interaction](../../subsystems/user-questions.md). _Avoid_: question type, render mode.

**timed question**: A question with a bounded foreground wait; when the window closes, the asking tool continues with the question still answerable instead of treating silence as an answer. Pending is not an empty answer, a refusal, or a permission. _Lives in_: [packages/interaction/user-questions](../../../packages/interaction/user-questions/README.md); [User Interaction](../../subsystems/user-questions.md). _Avoid_: async question, timeout question.

**continued question**: A timed question whose foreground window has closed, so its answer can now arrive only as a later reply message; it stays answerable until such a reply is admitted. It is a live question with no deadline, not a question that was withdrawn or never asked. _Lives in_: [packages/interaction/user-questions](../../../packages/interaction/user-questions/README.md); [User Interaction](../../subsystems/user-questions.md). _Avoid_: late question, stale question.

### Commands

**command shadowing**: The [shadowing](../../glossary.md#agent-scope) rule applied to commands: a command registered for one agent replaces the same-named global command for that agent alone, so discovery and execution see only the effective definition. It lets a plugin add or narrow a command without disturbing every other agent. _Lives in_: [packages/interaction/commands](../../../packages/interaction/commands/README.md); [Human Commands](../../subsystems/commands.md). _Avoid_: command override, per-agent command.

### Permissions

**permission preset**: A named bundle of the two enforcement knobs — sandbox mode and approval policy — offered to people as one Permissions choice, so selecting the preset writes both settings together. It is a presentation convenience: the preset layer owns no execution policy of its own. _Lives in_: [packages/interaction/permission-presets](../../../packages/interaction/permission-presets/README.md); [Permission Presets](../../subsystems/permission-presets.md). _Avoid_: permission level, mode.

**Auto preset**: A reserved preset that an optional integration publishes for the current session only, and whose availability disappears when that integration is disposed. It is never configurable and never becomes the default for new sessions. _Lives in_: [packages/interaction/permission-presets](../../../packages/interaction/permission-presets/README.md); [Permission Presets](../../subsystems/permission-presets.md). _Avoid_: automatic mode, smart preset.

**custom preset**: The derived value reported when a session's sandbox mode and approval policy match no available preset; it may label the current state but is never a switch target. Seeing it means the knobs were changed independently of any bundle. _Lives in_: [packages/interaction/permission-presets](../../../packages/interaction/permission-presets/README.md); [Permission Presets](../../subsystems/permission-presets.md). _Avoid_: other, mixed.

### Plan

**plan mode**: Per-agent collaboration state that, while active, puts deployment-written guidance into every model request and asks the agent to explore and design before executing. It is soft guidance, not a restriction: tools, sandbox, and approvals stay governed by their own settings. _Lives in_: [packages/plan/plan-mode](../../../packages/plan/plan-mode/README.md); [Plan Mode](../../subsystems/plan.md). _Avoid_: planning phase, read-only mode.

**pending plan selection**: A plan-mode change made while a turn is already open, held until the next accepted in-turn point before it becomes logged state. Until then the agent still runs under the previously logged state, and a selection that never reaches such a point is lost. _Lives in_: [packages/plan/plan-mode](../../../packages/plan/plan-mode/README.md); [Plan Mode](../../subsystems/plan.md). _Avoid_: queued change, uncommitted mode.

### Goal

**goal clear tombstone**: The durable record left when a goal is cleared, so ending the current objective does not erase its history or its revision. It records a cleared goal rather than the absence of any goal. _Lives in_: [packages/goal/goal](../../../packages/goal/goal/README.md); [Same-session goals](../../subsystems/goal.md). _Avoid_: delete, reset.

### Schedule

**scheduled task**: A Host-owned task that delivers a prompt into a session on a recurrence, kept independently of session activation so it survives restarts and stays inspectable without opening its target. Due deliveries arrive as ordinary follow-up messages, not as email, SMS, or push notifications. _Lives in_: [packages/schedule/schedule](../../../packages/schedule/schedule/README.md); [Host-wide Schedule](../../subsystems/schedule.md). _Avoid_: reminder, cron job, alarm.

**timing kind**: One of seven authoritative recurrence selectors a task may use — a delay, an absolute instant, a fixed interval, a daily or weekly wall-clock rule, a cron expression, or an RRULE — each keeping its own committed expression and zone. Normalization preserves committed occurrences and never converts one kind into another. _Lives in_: [packages/schedule/schedule](../../../packages/schedule/schedule/README.md); [Host-wide Schedule](../../subsystems/schedule.md). _Avoid_: schedule type, recurrence mode.

**delivery target**: Where a task's occurrences land: one existing session, one session reserved on first delivery and reused thereafter, or a new session per delivery. The target decides whether a task depends on a session existing when it is armed. _Lives in_: [packages/schedule/schedule](../../../packages/schedule/schedule/README.md); [Host-wide Schedule](../../subsystems/schedule.md). _Avoid_: destination, recipient.

**delivery receipt**: A saved record that a delivery was acknowledged into the target session's inbox, not that the model responded. History is read from these receipts and never from activating the target session. _Lives in_: [packages/schedule/schedule](../../../packages/schedule/schedule/README.md); [Host-wide Schedule](../../subsystems/schedule.md). _Avoid_: result, response, run log.

### Todo

**todo list**: One agent session's standing task list, replaced wholesale on every update, with entries carrying only a short content line and a three-state status. It is the agent's own working overview, distinct from a goal's objective and from a scheduled task. _Lives in_: [packages/todo/tool-todo](../../../packages/todo/tool-todo/README.md); [Todo](../../subsystems/todo.md). _Avoid_: task list, plan, checklist.

## Owned elsewhere

These terms are defined in the root [glossary](../../glossary.md); this context only uses them in the narrower senses below.

- [goal](../../glossary.md#goal) — here it names the durable objective alone; keeping work moving is the continuation driver's concern, not the goal's.
- [goal round](../../glossary.md#goal-round) — the continuation unit this context counts against a goal's round cap.
- [goal activation](../../glossary.md#goal) — the process-local permission to admit another goal round, which is why resume and fork need an explicit re-arm.
- [human command](../../glossary.md#human-command) — the unit plugins register, discover, and execute directly against an agent without spending a model message.
- [command plane](../../glossary.md#human-command) — the interface side that discovers, parses, dispatches, and renders human commands; command definitions stay plugin-owned.
- [goal command](../../glossary.md#human-command) — the `/goal` contributor that observes or mutates the current goal while the goal domain keeps every durable record.
