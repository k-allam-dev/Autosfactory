# Build Plan

The order to build in. The 9 planning tickets (see the Issues tab and `wayfinder-map.md`) decided what to build. This plan says what to do first.

The approach, decided in planning ticket 9: clone Orca (open source, MIT license) and add the features of this plan to the clone.

## Phase 0: Learn Orca and test the risky assumptions (no feature code)

1. **Clone and run Orca.** Get it running and launch Claude Code and Antigravity inside it. *Blocks everything.*
2. **Read Orca's code.** Write `docs/orca-notes.md` on how the Jira drawer, the agent dashboard, and agent launching and status tracking work, and where each can be extended. *Needs 1.*
3. **Set up the fork.** Add an upstream remote. Decide where our code lives (separate modules, as little change to Orca's core as possible) and how often to pull Orca updates. *Needs 1.*
4. **Run the early tests by hand.** Measure usage with Claude planning and Gemini Flash implementing on 3 real tickets. Check whether Antigravity's settings-file allow rules work on this machine. Check whether the Jira token works better through board tools or direct calls. If Flash fails, the implementer switches to Claude, which changes tickets 11 and 12. *Needs 1.*

## Phase 1: The core board

5. **Ticket state store.** Per-ticket state files on disk, the eight ticket states, and the separate agent status. *Needs 2.*
6. **The eight-column board.** Cards show the agent, the status label, and the flags. Extend Orca's agent dashboard. *Needs 5.*
7. **Jira intake.** Read-only sync every 30 minutes plus a refresh button, the token in the system keychain, alerts for changed tickets, and the Jira-down banner. *Needs 5 and 6.*

## Phase 2: Safe, controlled agents

8. **Safe launch and permission policy.** Stop Orca launching agents with the skip-permissions flag. Add the global and per-repo allow, ask, deny files, and the three-button prompt for Claude. *Needs 2 and 5.*
9. **Runner behavior.** One worktree per ticket with the branch named from the ticket key, up to 5 agents with a queue, pause and cancel, and agents coming back as interrupted after a restart. *Needs 5.*
10. **Review gate and PR.** A mandatory per-file diff review with line comments and a general feedback box. Approve opens the PR, and Request Changes sends it back. *Needs 6 and 9.*

## Phase 3: Two agents together

11. **Handoff note and plan phase.** Claude plans, then Antigravity implements from the approved plan. Agent switching only while stopped. *Needs 8 and 9.*
12. **Antigravity permissions.** The fixed pre-approved command list, plus the three-button prompt if the test in step 4 shows the allow rules work. *Needs 8 and 4.*

## Phase 4: Make agents cheaper

13. **Per-folder docs.** Short Claude-written docs per module in the target repos. *Needs 11.*
14. **Language server and small edits.** Add the TypeScript language server tool and the small-targeted-edits instruction, then measure whether agent steps drop. *Needs 13.*

## Why the tests come before the building

Step 4 comes before Phases 1 to 3 on purpose. If Flash cannot do the implementation, or Antigravity's allow rules do not work, it is better to find out before building the parts that depend on them.
