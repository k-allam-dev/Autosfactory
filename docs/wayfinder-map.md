# Wayfinder Map: Jira-to-Agents Ticket Board

A planning map for a local board that hands assigned Jira tickets to coding agents and tracks each ticket on a kanban board. All 9 planning tickets are resolved. The same tickets are in this repo's Issues tab.

## Destination

A v1 spec Kareem can build from: a local-first ticket board where Jira tickets assigned to him are handed to coding agents, each ticket tracked by a local state. Planning only: tickets resolve decisions, not code.

## Notes

- Domain: front-end development, many repos in one organization. Tickets range from tiny fixes (button text, vector indentation) to complex work.
- v1 runs on Kareem's own machine. Remote hosting comes later.
- Jira is read-only in v1. Kareem moves Jira statuses by hand. The agent never writes to Jira, and no PR-link comments.
- Standing preference: keep the board, state store, and agent runner as separate parts so they can move later.

## Decisions so far

- Per-ticket mode chosen at assignment: plan first, or direct implement.
- A human review gate with a per-file diff view before every PR, on every ticket.
- After approval the agent opens the PR itself, with title and description from the ticket. No Jira comment.
- The repo is mapped by Kareem per ticket, by picking a project folder before the agent starts.
- Local state only in v1, independent of Jira workflows (which differ per ticket type).
- Questions and answers happen in a chat box on the board card.
- A shared project-wide permission policy for all agents, with a way to promote a one-off approval into it.
- Kareem can move any ticket to any state manually. Moving to done or removing a ticket stops its agent.
- Agent controls: pause (resumable) and cancel (kills the agent and deletes its worktree).
- Crash and restart resilience: ticket and agent progress persist to disk.
- Agents are swappable per ticket (Claude, Antigravity, others).
- The board surface was planned as a local web kanban. Ticket 9 later moved the build into a clone of Orca (a desktop app).

## Ticket 1: State machine (resolved)

- Two separate pieces of status on every ticket: the ticket state (where the work is) and the agent status (how the agent is doing). They never mix.
- Ticket states: new, planning, plan ready for review, implementing, waiting on you, changes ready for review, PR open, done.
- Agent status values: idle, running, paused (you stopped it), interrupted (cut off or crashed), failed (hit an error). Shown as a label on the card, not as a column.
- A pending permission request is a separate flag on the card. It does not change the ticket state. A real question from the agent moves the ticket to waiting on you.
- Rejecting a plan sends the ticket back to planning. Rejecting changes sends it back to implementing with your feedback.
- Manual move: you can drag a ticket to any state, including skipping the review gate. The agent status resets to idle, and any running agent is stopped.

## Ticket 2: Agent runner design (resolved)

- Isolation: every ticket gets its own git worktree and branch, created from the repo mapped to the ticket. The branch name comes from the ticket key (for example PROJ-123). Two agents never touch the same files.
- Runner: a separate local process from the board. It starts one agent process per ticket through the native drivers, tracks each one, and reports events back to the board.
- Concurrency: up to five agents run in parallel. A sixth ticket waits in a queue with a visible queued label. The cap is a setting.
- Shared packages (npm): worktrees share one node_modules folder through a symlink. Installs through the symlink are blocked and the agent is told to ask first. When a dependency change is approved, that worktree gets its own private copy of node_modules and the install happens there, so other tickets keep the shared set.
- pnpm note: moving the repos to pnpm would make this simpler, since it keeps one global package store. That costs a migration and needs a test on one project first. An option, not part of v1.
- Pause: stops the agent but keeps the worktree, the conversation ID, and the handoff note, so resume continues the same session.
- Cancel: kills the process and deletes the worktree and branch, after a confirmation dialog.
- Persistence: each ticket has a small state file on disk with its ticket state, agent status, conversation ID, worktree path, and handoff note.
- Restart recovery: after the runner restarts, anything that was running is marked interrupted. Nothing resumes on its own. Kareem allows each agent to start again, one at a time, which avoids burning a usage limit by restarting all five at once.
- Cleanup: when a ticket is done, its worktree is kept until the PR is merged, then cleaned up. Cancel deletes it right away.

## Ticket 3: Agent portability (resolved)

- No common wrapper that translates between agents. Each agent gets its own native driver: the Claude Agent SDK for Claude, and the `agy` command line (stream-json output, conversation IDs for resume) for Antigravity.
- The agent window inside each card is agent-specific.
- Rule: shared by default, agent-specific only where the agent's own interface is involved. Shared: state machine, card header, git-based diff view, branch and PR creation, handoff note, worktree management. Agent-specific: chat window, permission prompts, session resume.
- Each driver maps its agent's output into a small set of board events (started, tool used, needs permission, finished, failed). Errors are not unified: the raw error text shows on the card.
- Integrations (Jira, Slack, GitHub): one shared allowed-tools list on the board. The board holds credentials once, in a local secrets file or the system keychain, and passes them to the agent process as environment variables per run. GitHub PR creation is done by the board using Kareem's existing git login, so agents need no GitHub access. Jira tokens are read-only.
- Permission levels in the board UI: allow once, allow for this session, allow for the whole project (stored in the shared policy). Claude uses a live permission callback. Antigravity has no live prompt in headless mode (see ticket 6).
- Antigravity headless facts: text, json, and stream-json output; resume by conversation ID or continue-latest; stdin streaming for multi-turn; allow, ask, deny rules in settings.json (deny beats ask beats allow); default run timeout of five minutes, so long tasks need a raised timeout; no native pause, so stop the process and resume the conversation later.

### Mixed-agent workflow

- An agent is chosen per ticket and per phase. Default: Claude plans, Antigravity implements from the approved plan.
- Each ticket has a handoff note. The planning agent writes the plan and exact steps into it. The implementing agent reads it first and leaves progress and next steps in it. This also supports resuming after an interruption.
- The card shows which agent owns the ticket. Different tickets can be driven by different agents at once.

## Ticket 4: Jira intake (resolved)

- Scope: tickets assigned to Kareem on two Jira boards (one scrum board and one operations board), in status Backlog, Selected for Development, or To Do. No issue type filter.
- Sync: a background job in the board itself, not an agent. It runs every 30 minutes and on a manual refresh button. It copies key, title, description, type, status, and link into local storage.
- Comments and attachments are not imported up front. The agent fetches them when it needs them.
- Jira status changed after import: the card stays where it is and shows a dismissible alert. The same applies if the ticket is reassigned away, with a stronger warning when an agent is mid-work.
- Ticket edited after import: a dismissible alert. The agent keeps working from the original text. A button sends the updated text to the agent as a new message.
- Login: one personal Jira API token in the system keychain, read-only. Agents reach Jira through read-only tools the board hands them, or direct calls to Jira's web API with the same token. Which works better is tested in the first build step.
- Jira down or token expired: the board keeps showing the last good data, shows a banner, and retries on the next cycle.
- Claude usage limit reached: the card shows a usage limit label with the reset time when Claude reports one. The agent is marked interrupted, not failed, and its handoff note is saved. Kareem chooses to wait and resume after the reset, or switch that ticket to Antigravity.

## Ticket 5: Board and ticket card design (resolved)

- Layout: a kanban with eight columns matching the ticket states.
- Card face: ticket key and title, the owning agent, the agent status label, and small flags for anything needing attention: a permission request, a Jira change alert, a usage limit, queued. No progress hint or timer in v1.
- Opening a card: a side panel slides in over the board. Four areas: ticket details and controls (mode, agent, pause, cancel), the chat box, the handoff note and plan, and a diff tab.
- Diff review: no agent involved. The board runs a plain `git diff` in the ticket's worktree against the branch it started from, after staging everything so new files appear too. A file list on the left shows each file with a reviewed mark, and changes are side by side on the right.
- Review actions: line-level comments and one general feedback box. Approve opens the PR. Request Changes sends the feedback to the agent and moves the ticket back to implementing.
- Chat box: a message thread with the agent's questions highlighted. Kareem can paste screenshots and send a new instruction at any time, even while the agent is running.
- Agent controls: pause and cancel at the top of the panel, with a confirmation on cancel. A permission request shows as a banner on the card and in the panel with three buttons.
- Switching agents is allowed only while the agent is stopped, paused, or interrupted.

## Ticket 6: Shared permission policy (resolved)

- Format: plain files on disk that can be edited by hand. One global file and one file per repo, each with allow, ask, and deny lists. Precedence is deny over ask over allow. The board translates the policy into each agent's native format when a ticket starts.
- Starting baseline. Allow: reads and edits inside the worktree, git status, git diff, npm test, npm run lint. Ask: anything that installs packages, uses the network, or runs other shell commands. Deny: anything destructive, such as deleting outside the worktree, force-pushing, or reading secrets files.
- Two layers: a global baseline and a per-repo list that grows as rules are promoted. The deny list is global and always wins.
- Promote flow: answering a permission request with allow once or allow for this session offers a make permanent link. The board shows the exact, narrowest rule before saving (for example `npm run build` rather than all npm commands). A rule can never be promoted past the deny list, and applies only to that ticket's repo.
- Antigravity finding: in headless mode there is no live approval, and an action that needs permission is soft-denied while the run continues. Open reports say headless mode can ignore allow rules in the settings file even when the rule loads. The skip-permissions flag removes every guardrail and is not used for real tickets.
- Antigravity fallback: it implements only inside the worktree, with a fixed pre-approved command list. Anything outside it stops the ticket and asks Kareem. Whether the settings-file allow rules work on his machine is tested in the first build step.
- Claude keeps all three buttons through its live permission callback.

## Ticket 7: Antigravity plan and usage (resolved)

- Antigravity has no subscription of its own. It rides on the Google AI plan. Google AI Pro is about $20 a month, with limits refreshed every five hours up to a weekly cap, and no published fixed numbers. Reports say the limits can be tight.
- An API key from AI Studio does not unlock smarter models under the subscription, and is billed separately. Decision: skip the API key route for v1.
- Decision: Claude plans, Antigravity with a Gemini Flash model implements from the approved plan. This is a bet, not proven. Implementation is the expensive phase, and planning is expected to cost much less.
- First build step: run Flash on a few real tickets from Claude-written plans, and measure usage for plans versus implementations. If Flash fails, switch the implementing agent to Claude.
- The board shows a quota-hit label on the card, using the interrupted or failed agent status.
- Per-folder docs: each module or folder in the target repos gets a short doc written by Claude (what it does, key files, how it connects). Planning agents read these first instead of exploring code. The agent updates the doc when it finishes a ticket.

## Ticket 8: Faster code search and editing (resolved)

- Kept: a TypeScript language server tool for both agents (jump to definition, list usages, see type errors in the same turn).
- Kept: the per-folder docs from ticket 7.
- Kept: a small-targeted-edits instruction (change only needed lines, do not rewrite whole files, do not re-read files already seen). A low-risk experiment, easy to remove.
- Dropped: file hints in tickets, because Kareem does not write the tickets. May be an optional note.
- Tested in the first build step: whether the language server tool and the edit instruction reduce agent steps.

## Ticket 9: Build on Orca (resolved)

- Decision: clone Orca (open source, MIT license, Electron, TypeScript, React, pnpm) and add this plan's features to the clone, instead of building the board from scratch.
- Already in Orca, to reuse: parallel agents each in their own git worktree, support for Claude Code and Antigravity, diff review with line comments sent back to the agent, commit and push, Claude usage tracking, a Jira link drawer, and an experimental agent dashboard shaped like a kanban.
- To add, unique to this plan: automatic read-only Jira intake, the eight ticket states with the separate agent status label, the mandatory review gate before every PR (the board opens the PR), the shared permission policy files, the Claude-plans and Antigravity-implements handoff note, and the restart rule.
- Differences to resolve in the fork: Orca launches agents with skip-permissions flags by default, which this plan does not allow. Orca runs agents in terminals, so structured events, pause and resume, and permission callbacks may need extra work. The board becomes part of a desktop app.
- Risk: Orca changes quickly, so the fork can fall behind. Keep added features in separate modules, touch Orca's core files as little as possible, and pull in upstream updates on a regular schedule.
- First step before any feature work: clone and run Orca, then read how the Jira drawer, the agent dashboard, and agent launching and status tracking are built, and decide how each gets extended. The usage, permission, and language server tests still happen after that.

## Not yet specified

- Remote hosting: how to run without Kareem's laptop on.
- Automating Jira status transitions per ticket type.
- Extra agent-card telemetry beyond what is specified above.

## Out of scope

- Writing to Jira in v1.
- Multi-user or team use.
