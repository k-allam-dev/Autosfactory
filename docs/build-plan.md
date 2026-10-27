# Build Plan

The order to build in. The 9 planning tickets (see `wayfinder-map.md`) decided what to build. This plan says what to do first.

The approach, decided in planning ticket 9: fork Orca (open source, MIT license) and add the features of this plan to the fork.

**Tracking now happens in the fork:** [k-allam-dev/orca](https://github.com/k-allam-dev/orca/). Each phase is an issue ([#10](https://github.com/k-allam-dev/orca/issues/10), [#11](https://github.com/k-allam-dev/orca/issues/11), [#12](https://github.com/k-allam-dev/orca/issues/12), [#13](https://github.com/k-allam-dev/orca/issues/13), [#14](https://github.com/k-allam-dev/orca/issues/14)) with its steps as sub-issues. Step N is issue #(N+14) for steps 1 to 14. Research steps are 4a ([#29](https://github.com/k-allam-dev/orca/issues/29)), 4b ([#30](https://github.com/k-allam-dev/orca/issues/30)) and 8a ([#31](https://github.com/k-allam-dev/orca/issues/31)). This file is a snapshot of the order; the issues hold the details and the current state.

Two later decisions are built into this order:

- **No skip-permissions, and isolation per ticket.** Agents run with a real permission policy (step 8) *and* inside an isolated environment per ticket (step 8a), because a rule list alone does not confine an agent to a folder.
- **Tests before building.** Step 4 and the research steps come before the parts that depend on them. If Flash cannot do the implementation, Antigravity's allow rules do not work, or the isolated environment is not workable, it is better to find out first.

## Phase 0: Learn Orca and test the risky assumptions (no feature code) ([#10](https://github.com/k-allam-dev/orca/issues/10))

1. **Clone and run Orca** ([#15](https://github.com/k-allam-dev/orca/issues/15)). Get it running and launch Claude Code and Antigravity inside it. *Blocks everything.* *Status: App runs; waiting for the agent and Jira tests.*
2. **Read Orca's code** ([#16](https://github.com/k-allam-dev/orca/issues/16)). Write notes on how the Jira drawer, the agent dashboard, and agent launching and status tracking work, and where each can be extended. *Needs 1.* *Status: Notes written (`orca-notes.md`).*
3. **Set up the fork** ([#17](https://github.com/k-allam-dev/orca/issues/17)). Add an upstream remote. Decide where our code lives (separate modules, as little change to Orca's core as possible) and how often to pull Orca updates. *Needs 1.* *Status: Rules written (`docs/autosfactory/fork-rules.md` in the fork).*
4. **Run the early tests by hand** ([#18](https://github.com/k-allam-dev/orca/issues/18)). Measure usage with Claude planning and Gemini Flash implementing on 3 real tickets. Check whether Antigravity's settings-file allow rules work on this machine. Check whether the Jira token works better through board tools or direct calls. If Flash fails, the implementer switches to Claude, which changes steps 11 and 12. *Needs 1.* *Status: Not started.*
4a. **Research Antigravity's ACP route** ([#29](https://github.com/k-allam-dev/orca/issues/29)). Find out whether Orca's ACP route gives Antigravity live permission prompts. Result: no Windows build at this commit, so on this PC the fixed command list stays the base. *Status: Read-only part done; needs a real run on macOS or Linux (or inside WSL).*
4b. **Research the exact details of each permission prompt** ([#30](https://github.com/k-allam-dev/orca/issues/30)). What each Claude and Antigravity prompt contains and what each answer stores. *Status: Read-only part done; needs a real approve and deny run.*

## Phase 1: The core board ([#11](https://github.com/k-allam-dev/orca/issues/11))

5. **Ticket state store** ([#19](https://github.com/k-allam-dev/orca/issues/19)). Per-ticket state files on disk, the eight ticket states, and the separate agent status. *Needs 2.* *Status: Not started.*
6. **The eight-column board** ([#20](https://github.com/k-allam-dev/orca/issues/20)). Cards show the agent, the status label, and the flags. A new ticket-based board that reuses Orca's card and column components. *Needs 5.* *Status: Not started.*
7. **Jira intake** ([#21](https://github.com/k-allam-dev/orca/issues/21)). Read-only sync every 30 minutes plus a refresh button, the token in the system keychain, alerts for changed tickets, and the Jira-down banner. Orca already has the Jira client, so this is mostly the background job and alerts. *Needs 5 and 6.* *Status: Not started.*

## Phase 2: Safe, controlled agents ([#12](https://github.com/k-allam-dev/orca/issues/12))

8a. **Research an isolated environment per ticket** ([#31](https://github.com/k-allam-dev/orca/issues/31)). Prove that each ticket's agent can run in its own WSL2-based environment that cannot reach the rest of the PC, and choose between a distro per ticket, a shared hardened distro with per-ticket users, or a container per ticket. *Needs Orca running (step 1). Blocks 8 and 9.* *Status: Not started.*
8. **Safe launch and permission policy** ([#22](https://github.com/k-allam-dev/orca/issues/22)). Stop Orca launching agents with the skip-permissions flag. Add the global and per-repo allow, ask, deny files, deny rules for secrets, and the three-button prompt for Claude. Runs inside the isolated environment from 8a. *Needs 2, 5, 4a, 4b and 8a.* *Status: Not started.*
9. **Runner behavior** ([#23](https://github.com/k-allam-dev/orca/issues/23)). One worktree (and isolated environment) per ticket with the branch named from the ticket key, up to 5 agents with a queue, pause and cancel, and agents coming back as interrupted after a restart. *Needs 5 and 8a.* *Status: Not started.*
10. **Review gate and PR** ([#24](https://github.com/k-allam-dev/orca/issues/24)). A mandatory per-file diff review with line comments and a general feedback box. Approve opens the PR, and Request Changes sends it back. *Needs 6 and 9.* *Status: Not started.*

## Phase 3: Two agents together ([#13](https://github.com/k-allam-dev/orca/issues/13))

11. **Handoff note and plan phase** ([#25](https://github.com/k-allam-dev/orca/issues/25)). Claude plans, then Antigravity implements from the approved plan. Agent switching only while stopped. *Needs 8 and 9.* *Status: Not started.*
12. **Antigravity permissions** ([#26](https://github.com/k-allam-dev/orca/issues/26)). The fixed pre-approved command list, plus the three-button prompt if the test in step 4 shows the allow rules work. *Needs 8 and 4.* *Status: Not started.*

## Phase 4: Make agents cheaper ([#14](https://github.com/k-allam-dev/orca/issues/14))

13. **Per-folder docs** ([#27](https://github.com/k-allam-dev/orca/issues/27)). Short Claude-written docs per module in the target repos. *Needs 11.* *Status: Not started.*
14. **Language server and small edits** ([#28](https://github.com/k-allam-dev/orca/issues/28)). Add the TypeScript language server tool and the small-targeted-edits instruction, then measure whether agent steps drop. *Needs 13.* *Status: Not started.*
