
This workflow guides AI agent development in this repository.

<!-- OPENSPEC:START -->

**Why this workflow?** This combines three complementary tools for structured, multi-agent development:
1. **OpenSpec** handles the planning phase — ensuring human and AI align on *what* to build via research, proposals, and specs before any code is written. 
2. **Beads** handles the execution phase — tracking issues with dependency graphs so agents know *what's ready to work on* and *what's blocked*. We use **beads-rust** (`br`), Jeffrey Emanuel's Rust fork of Steve Yegge's original beads which freezes the SQLite + JSONL architecture, and **beads-viewer** (`bv`),  which adds graph-aware AI guidance via `bv --robot-*` commands. 
3. **Coordination yamls** handle multi-agent coordination — explicitly tracking which agent owns which files to prevent conflicts across git worktrees. 
Together: openspec provides project specs, beads provides issue tracking, and coordination yamls provide multi-agent coordination.

### Workflow preamble:

- **OpenSpec usage**: We use OpenSpec mostly according to standard guidance, but diverge in the following ways:
	- No `tasks` markdown files -- tasks are now handled completely by beads.
	- Each change should only target a single domain.
		- Each delta spec `prd.md` should explicitly state the single domain it targets at the top of the file (e.g. `"Target domain: <domain-name>"`)
	- A single, flat, 'master' specs file: `openspec/specs/prd.md`.
		- We simplify and consolidate the standard many `openspec/specs/<domain>/spec.md` files into a single `prd.md` that is structured by 'domain' headers.
		- The headers in the 'master' prd should be:
			- `##` for each domain
			- `###` for requirements within a domain
			- `####` for scenarios within a requirement
		- Because of this, it's _crucial to keep the delta `prd.md` of each openspec change relatively small and clearly scoped (single-domain)_ for clean merge back into the 'master' `prd.md`.
- **Human-agent conversation initiation**: Within this workflow, I'll typically initiate conversations with something like: "Hey, you're `<agent-name>`, excited to be working with you on this project! I'd like you to help me implement `<change-name>`. `<change-description>`..."
- **Commands to use**:  When in this workflow, you should generally make extensive use of openspec, beads-rust, beads-viewer, and the `.coordination/` directory.
	- **3rd-party CLIs**: CLIs for openspec, beads-rust, and beads-viewer are already installed. Notes on the CLIs are in sections in this doc further below. See the source code in `~/agent_tool_repos/` if you ever have questions or issues when using these tools.
	- **Slash commands**: Additionally, you may find it helpful to use `/feature-dev` and `/requesting-code-review` commands at certain times. When these commands may be useful is detailed below.
- **Assigned agent name for coordination**: You will use the name I've assigned you to coordinate with me and other agents working on this project. In the `.coordination/` dir, you will find a yaml file referencing this assigned name: `agent-<your_name>.yaml`, as well as other yaml files referencing other agents by the names I've assigned them. You should ONLY EVER update your yaml file, NEVER the yaml of the other agents.
- **"Agent-step" definition**: An "agent-step" is considered to be any artifact generation in the openspec schema, any update or creation of a beads issue, and any individual file edit.
- **Example coordination yaml template:**
```yaml
agent: agent-N  # agent name
worktree: etl-worktrees/agent-N  # git worktree directory
current_change: null  # folder change-name
status: idle  # idle | researching | planning | implementing | blocked | review-ready
beads_issue: null # id of beads issue currently working on
owned_paths: []  # list of files currently being edited, blocked to other agents
blocked_on: null  # blocked on details, if any (e.g. blocking beads issue id, blocking other change-name, etc.)
needs_human: false  # whether edits from this change are ready for human review
last_updated: null  # YYYY-MM-DD_HH-MM-SSZ (UTC) of last update within this change
notes: ""  # Any additional notes helpful for a human or agent to know
```

### Working on a change

After I've initiated a conversation that triggers this workflow:
1. **Create feature branch**: 
	1. Git pull main branch
	2. `br sync --import-only`
	3. Checkout a feature branch for this change.
2. **Coordinate**: 
	1. When starting a change, and at each agent-step while working on a change, you should update your own coordination .yaml file as necessary, *in the main git worktree, not in your own git worktree* (for shared visibility with all agents). 
	2. Then, before proceeding onto your next agent-step, first consult all other agents' yaml files in `.coordination/` to ensure no potential conflicts with other agents; if no conflicts, proceed to "Start change".
	3. If you notice a conflict with another agent, then start a timer, and try to coordinate with that agent via the `notes` field in your yamls (e.g. tell that agent what you're blocked on, what you need, and to respond). If you're a blocking agent, respond in your `notes` field to the blocked agent. Do not make any project updates while there is still a conflict and/or you are still blocked. If after 10 minutes you are still blocked, message me and let me know.
3. **Start change**: Use openspec's `explore`, `new`, `continue`, and `ff` `/opsx:` commands, each as helpful, to start a change and generate artifacts for this change.
	1. Additionally, feel free to use `feature-dev/` here as helpful.
4. **Go from change specs -> beads issues**: Once you have generated the final openspec 'specs' artifact for this change, turn this into beads issues using `br*` commands. *Remember, we are not using 'tasks' within openspec, and instead replacing this with beads*.
	1. Use beads-viewer `bv*` commands as and when helpful alongside `br` to organize beads issues.
5. **Implement and verify change-related updates**: Work on the beads issues until you believe you have implemented and verified all updates needed to finalize this change -- i.e. you've closed all beads issues related to this change.
	1. Use `/opsx:` `verify` alongside `br` and `bv` commands here as helpful. *Remember, we are not using 'tasks' within openspec, and instead use beads to manage tasks/issues under a change*, so we _do not_ use `/opsx:` `apply` here.

#### Discovering additional tasks when working on a change

##### Tangential issues (not blocking)

When you discover issues that don't block your current work (e.g., "this docstring is outdated", "this function could be refactored", etc.):

1. **Quick capture with beads**, e.g.  
```bash
   br q "Refactor storage base class for better typing"
   # Created: etl-d4e5f6
```

2. **Add labels for categorization**, e.g. `br label add etl-d4e5f6 tech-debt discovered`

3. **Sync so others can see it**: `br sync --flush-only`

4. **Do NOT work on it now** — continue with your current change. Tangential issues go in the backlog for future prioritization.

##### Blocking issues (must fix first)

When you discover something that blocks your current work (e.g., "the base class I need doesn't exist", "there's a bug in the dependency I'm using", etc.):

1. **Create a blocking issue**, e.g.
```bash
   br create "Fix fsspec credential handling" \
     --type bug \
     --priority 1 \
     --description "Discovered while implementing ingestion"
   # Created: etl-g7h8i9
```

2. **Add dependency**, e.g.
```bash
   br dep add etl-<your-current-task> etl-g7h8i9
```

3. **Decide how to proceed**:
   - If the blocker is small and in your domain: fix it and continue.
   - If the blocker is owned by another agent: follow the "if you notice a conflict with another agent" instructions above.
   - If the blocker is large or outside your domain: stop and message me

#### Context Switching

If I ask you to pause your current change and work on something else:
1. Update your coordination yaml: `status: paused`, add note explaining state.
2. `br sync --flush-only` to save any beads updates.
3. Do NOT archive — the change is incomplete.
4. Start the new change as normal.
5. When returning, read your own yaml `notes` to restore context.
#### When to stop and ask

In addition to any already mentioned occasions on which you should stop what you're doing and ask for my input (e.g. if you ever think of removing or deleting anything, as mentioned in "Practice to follow" -> "Rules" -> "2."),  you MUST always stop and ask me before proceeding for:

1. **Design concerns**: You see a flaw, ambiguity, or potential issue with my proposal that could lead to: 
	1. Incorrect implementation
	2. Security/safety problems
	3. Significant scope creep (change targets multiple domains or is much larger than described)
	4. Architectural conflict with existing code 
2. **Ambiguous requirements**: The spec or my request is unclear enough that you'd be guessing at intent.
3. **Technical blockers**: Something is technically impossible or requires changes outside your assigned scope.

### Completing a change

1. **Request sub-agent review**: After you believe you've finished "Working on a change" for one particular change (closed all beads issues related to this change), run `/requesting-code-review` to get a new subagent to do code review on the changes you've made, and only when these have been accepted in a loop between you and this subagent, then present the changes for me to review.
2. **Request my review:** Request my review on all the updates you've made for this change. We may have a continued conversation at this point, until I'm happy and I tell you I've accepted the updates.
3. **Sync and archive change**: 
	1. Use `/opsx:` `sync` to merge delta specs into main specs (`openspec/specs/prd.md).` Carefully check this `prd.md` to ensure the merge has been done properly.
	2. Use `/opsx:` `archive` to archive the change.
	3. Update (i.e. "reset") your coordination yaml file.
	4. Ask me to do a final review and if you or I should make a git commit.

### Additional workflow notes

**Where the OpenSpec truth lives**
- Current system truth: `openspec/specs/prd.md`
- Project-level config (defaults + global context + per-artifact rules): `openspec/config.yaml`
- OpenSpec schema to use: `openspec/schemas/*.yaml`
- Proposed work (one folder per change): `openspec/changes/<change-name>/**`

**Coordination rules**:
- Only ONE agent may own a file at a time, made explicit in `owned_paths` in the coordination yamls
- After ANY beads update: `br sync --flush-only`

---

Keep this managed block so `openspec update` can refresh OpenSpec-related instructions when needed.

<!-- OPENSPEC:END -->

---

### openspec quick reference

| Command              | Purpose                                             |
| -------------------- | --------------------------------------------------- |
| `/opsx:explore`      | Think through ideas before committing to a change   |
| `/opsx:new`          | Start a new change                                  |
| `/opsx:continue`     | Create the next artifact based on dependencies      |
| `/opsx:ff`           | Fast-forward: create all planning artifacts at once |
| `/opsx:apply`        | Implement tasks from the change                     |
| `/opsx:verify`       | Validate implementation matches artifacts           |
| `/opsx:sync`         | Merge delta specs into main specs                   |
| `/opsx:archive`      | Archive a completed change                          |
| `/opsx:bulk-archive` | Archive multiple changes at once                    |
| `/opsx:onboard`      | Guided tutorial through the complete workflow       |

---

### beads-rust quick reference

Issue Lifecycle

| Command  | Description              | Example                              |
| -------- | ------------------------ | ------------------------------------ |
| `init`   | Initialize workspace     | `br init`                            |
| `create` | Create issue             | `br create "Title" -p 1 --type bug`  |
| `q`      | Quick capture (ID only)  | `br q "Fix typo"`                    |
| `show`   | Show issue details       | `br show bd-abc123`                  |
| `update` | Update issue             | `br update bd-abc123 --priority 0`   |
| `close`  | Close issue              | `br close bd-abc123 --reason "Done"` |
| `reopen` | Reopen closed issue      | `br reopen bd-abc123`                |
| `delete` | Delete issue (tombstone) | `br delete bd-abc123`                |

Querying

| Command | Description | Example |
|---------|-------------|---------|
| `list` | List issues | `br list --status open --priority 0-1` |
| `ready` | Actionable work | `br ready` |
| `blocked` | Blocked issues | `br blocked` |
| `search` | Full-text search | `br search "authentication"` |
| `stale` | Stale issues | `br stale --days 30` |
| `count` | Count with grouping | `br count --by status` |

Dependencies

| Command | Description | Example |
|---------|-------------|---------|
| `dep add` | Add dependency | `br dep add bd-child bd-parent` |
| `dep remove` | Remove dependency | `br dep remove bd-child bd-parent` |
| `dep list` | List dependencies | `br dep list bd-abc123` |
| `dep tree` | Dependency tree | `br dep tree bd-abc123` |
| `dep cycles` | Find cycles | `br dep cycles` |

Labels

| Command | Description | Example |
|---------|-------------|---------|
| `label add` | Add labels | `br label add bd-abc123 backend urgent` |
| `label remove` | Remove label | `br label remove bd-abc123 urgent` |
| `label list` | List issue labels | `br label list bd-abc123` |
| `label list-all` | All labels in project | `br label list-all` |

Comments

| Command | Description | Example |
|---------|-------------|---------|
| `comments add` | Add comment | `br comments add bd-abc123 "Found root cause"` |
| `comments list` | List comments | `br comments list bd-abc123` |

Sync & System

| Command | Description | Example |
|---------|-------------|---------|
| `sync` | Sync DB ↔ JSONL | `br sync --flush-only` |
| `doctor` | Run diagnostics | `br doctor` |
| `stats` | Project statistics | `br stats` |
| `config` | Manage config | `br config --list` |
| `upgrade` | Self-update | `br upgrade` |
| `version` | Show version | `br version` |

Global Flags

| Flag               | Description                        |
| ------------------ | ---------------------------------- |
| `--json`           | JSON output (machine-readable)     |
| `--quiet` / `-q`   | Suppress output                    |
| `--verbose` / `-v` | Increase verbosity (-vv for debug) |
| `--no-color`       | Disable colored output             |
| `--db <path>`      | Override database path             |

---

### beads-viewer quick reference

Using `bv` as an AI sidecar:

`bv` is a graph-aware triage engine for Beads projects (.beads/beads.jsonl). Instead of parsing JSONL or hallucinating graph traversal, use robot flags for deterministic, dependency-aware outputs with precomputed metrics (PageRank, betweenness, critical path, cycles, HITS, eigenvector, k-core).

**⚠️ CRITICAL: Use ONLY `--robot-*` flags. Bare `bv` launches an interactive TUI that blocks your session.**

**`bv --robot-triage` is your single entry point**: it returns everything you need in one call:
- `quick_ref`: at-a-glance counts + top 3 picks
- `recommendations`: ranked actionable items with scores, reasons, unblock info
- `quick_wins`: low-effort high-impact items
- `blockers_to_clear`: items that unblock the most downstream work
- `project_health`: status/type/priority distributions, graph metrics
- `commands`: copy-paste shell commands for next steps

**`bv --robot-next` is minimal**: just the single top pick + claim command.

Use token-optimized output (TOON) for lower LLM context usage:
`bv --robot-triage --format toon`

Planning

| Command            | Returns                                         |
| ------------------ | ----------------------------------------------- |
| `--robot-plan`     | Parallel execution tracks with `unblocks` lists |
| `--robot-priority` | Priority misalignment detection with confidence |

Graph Analysis

| Command                                         | Returns                                                                                                                              |
| ----------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| `--robot-insights`                              | Full metrics: PageRank, betweenness, HITS (hubs/authorities), eigenvector, critical path, cycles, k-core, articulation points, slack |
| `--robot-label-health`                          | Per-label health: `health_level` (healthy\|warning\|critical), `velocity_score`, `staleness`, `blocked_count`                        |
| `--robot-label-flow`                            | Cross-label dependency: `flow_matrix`, `dependencies`, `bottleneck_labels`                                                           |
| `--robot-label-attention [--attention-limit=N]` | Attention-ranked labels by: `(pagerank × staleness × block_impact) / velocity`                                                       |

History & Change Tracking

| Command                               | Returns                                                                                                |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| `--robot-history`                     | Bead-to-commit correlations: `stats`, `histories` (per-bead events/commits/milestones), `commit_index` |
| `--robot-diff --diff-since %3Cref%3E` | Changes since ref: new/closed/modified issues, cycles introduced/resolved                              |

Other Commands

| Command                                             | Returns                                                            |
| --------------------------------------------------- | ------------------------------------------------------------------ |
| `--robot-burndown <sprint>`                         | Sprint burndown, scope changes, at-risk items                      |
| `--robot-forecast <id\|all>`                        | ETA predictions with dependency-aware scheduling                   |
| `--robot-alerts`                                    | Stale issues, blocking cascades, priority mismatches               |
| `--robot-suggest`                                   | Hygiene: duplicates, missing deps, label suggestions, cycle breaks |
| `--robot-graph [--graph-format=json\|dot\|mermaid]` | Dependency graph export                                            |
| `--export-graph <file.html>`                        | Self-contained interactive HTML visualization                      |

Scoping & Filtering

| Command                                     | Purpose                                 |
| ------------------------------------------- | --------------------------------------- |
| `bv --robot-plan --label backend`           | Scope to label's subgraph               |
| `bv --robot-insights --as-of HEAD~30`       | Historical point-in-time                |
| `bv --recipe actionable --robot-plan`       | Pre-filter: ready to work (no blockers) |
| `bv --recipe high-impact --robot-triage`    | Pre-filter: top PageRank scores         |
| `bv --robot-triage --robot-triage-by-track` | Group by parallel work streams          |
| `bv --robot-triage --robot-triage-by-label` | Group by domain                         |

#### Understanding robot output

**All robot JSON includes:**
- `data_hash` — Fingerprint of source beads.jsonl (verify consistency across calls)
- `status` — Per-metric state: `computed|approx|timeout|skipped` + elapsed ms
- `as_of` / `as_of_commit` — Present when using `--as-of`; contains ref and resolved SHA

**Two-phase analysis:**
- **Phase 1 (instant):** degree, topo sort, density — always available immediately
- **Phase 2 (async, 500ms timeout):** PageRank, betweenness, HITS, eigenvector, cycles — check `status` flags

**For large graphs (>500 nodes):** Some metrics may be approximated or skipped. Always check `status`.

#### `jq` quick reference

```
bv --robot-triage | jq '.quick_ref'                        # At-a-glance summary
bv --robot-triage | jq '.recommendations[0]'               # Top recommendation
bv --robot-plan | jq '.plan.summary.highest_impact'        # Best unblock target
bv --robot-insights | jq '.status'                         # Check metric readiness
bv --robot-insights | jq '.Cycles'                         # Circular deps (must fix!)
bv --robot-label-health | jq '.results.labels[] | select(.health_level == "critical")'
```

**Performance:** Phase 1 instant, Phase 2 async (500ms timeout). Prefer `--robot-plan` over `--robot-insights` when speed matters. Results cached by data hash.

Use `bv` instead of parsing beads.jsonl -- it computes PageRank, critical paths, cycles, and parallel tracks deterministically.