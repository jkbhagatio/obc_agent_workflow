
This workflow guides AI agent development in this repository.

<!-- OPENSPEC:START -->

**Why this workflow?** This combines three complementary tools for structured, multi-agent
development:

1. **OpenSpec** handles the planning phase—ensuring human and AI align on *what* to build via
   research, proposals, and specs before any code is written.
2. **Beads** handles the execution phase—tracking issues with dependency graphs so agents know
   *what's ready to work on* and *what's blocked*. Use **beads-rust** (`br`), Jeffrey Emanuel's
   Rust fork of Steve Yegge's original beads (which freezes the SQLite + JSONL architecture), and
   **beads-viewer** (`bv`), which adds graph-aware AI guidance via `bv --robot-*` commands.
3. **Coordination yamls** handle multi-agent coordination—explicitly tracking which agent owns
   which files to prevent conflicts across git worktrees.

Together: OpenSpec provides project specs, beads provides issue tracking, and coordination yamls
provide multi-agent coordination.

See the quick reference sections below for details on OpenSpec, beads-rust, and beads-viewer
commands.

### Key definitions

- **PRD / spec**: These terms are used interchangeably throughout this document. Both refer to
  product requirements documents that specify what to build.
- **Domain**: A logical grouping of related functionality in the project. For a Python project,
  think of a domain as somewhere between the level of a single module and a single sub-package
  directory, inclusive. Each domain gets its own `## Domain:` header in the master PRD.
- **Main worktree**: The main, shared repository directory (e.g., `etl/`). All agents can see
  files here. Coordination yamls live here for shared visibility.
- **Agent worktree**: An agent's isolated git worktree (e.g., `etl-worktrees/agent-claudia/`).
  Each agent works in their own worktree to avoid conflicts.
- **Agent-step**: Any artifact generation in the OpenSpec schema, any update or creation of a
  beads issue, or any individual file edit.

### Workflow preamble

- **OpenSpec usage**: Use OpenSpec mostly according to standard guidance, but diverge in the
  following ways:
    - Do not create `tasks` markdown files—beads handles all task tracking.
    - Target only a single domain per change.
        - State the single target domain at the top of each delta spec (e.g.,
          `"Target domain: <domain-name>"`).
    - Use a single, flat, "master" PRD file: `openspec/specs/prd.md`.
        - This consolidates the standard `openspec/specs/<domain>/spec.md` files into one master
          PRD structured by domain headers.
        - Use these header levels in the master PRD:
            - `##` for each domain.
            - `###` for requirements within a domain.
            - `####` for scenarios within a requirement.
        - Keep each delta PRD small and clearly scoped (single-domain) for clean merge back into
          the master PRD.
- **Human-agent conversation initiation**: Expect conversations to start with something like:
  "Hey, you're `<agent-name>`, excited to be working with you on this project! I'd like you to
  help me implement `<change-name>`. `<change-description>`..."
- **Commands to use**: Make extensive use of OpenSpec, beads-rust, beads-viewer, and the
  `.coordination/` directory throughout this workflow.
    - **3rd-party CLIs**: The CLIs for OpenSpec, beads-rust, and beads-viewer are already
      installed. See the source code in `~/agent_tool_repos/` for questions or issues.
    - **Slash commands**: Use `/feature-dev` and `/requesting-code-review` commands as detailed
      below.
- **Assigned agent name for coordination**: Use the assigned name to coordinate with the human
  and other agents. Find the coordination yaml at `.coordination/agent-<your_name>.yaml`. Only
  ever update your own yaml file, never the yamls of other agents.
- **Example coordination yaml template:**

```yaml
agent: agent-N  # agent name
worktree: etl-worktrees/agent-N  # agent's git worktree directory
current_change: null  # folder change-name
status: idle  # idle | researching | planning | implementing | blocked | review-ready
beads_issue: null # id of beads issue currently working on
owned_paths: []  # list of files currently being edited, blocked to other agents
blocked_on: null  # blocked on details, if any (e.g. blocking beads issue id, other change-name)
needs_human: false  # whether edits from this change are ready for human review
last_updated: null  # YYYY-MM-DD_HH-MM-SSZ (UTC) of last update within this change
notes: ""  # any additional notes helpful for a human or agent to know
```

### Working on a change

After the human initiates a conversation that triggers this workflow:

1. **Create feature branch**:
    - Pull the main branch.
    - Run `br sync --import-only`.
    - Check out a feature branch for this change.
2. **Coordinate**:
    - When starting a change, and at each agent-step, update your coordination yaml file *in the
      main worktree* (not your agent worktree) for shared visibility with all agents.
    - Before proceeding to the next agent-step, consult all other agents' yaml files in
      `.coordination/` to check for potential conflicts. If no conflicts, proceed to "Start
      change".
    - If a conflict exists with another agent, start a timer and coordinate via the `notes` field
      in your yamls (e.g., state what you're blocked on, what you need, and request a response).
      If you are the blocking agent, respond in your `notes` field. Do not make any project
      updates while a conflict exists or while blocked. If still blocked after 10 minutes,
      message the human for guidance on how to proceed.
3. **Start change**: Use OpenSpec's `/opsx:` commands (`explore`, `new`, `continue`, `ff`) to
   start a change and generate artifacts.
    - Optionally use `/feature-dev` here as helpful.
4. **Go from change specs to beads issues**: Once the final OpenSpec "specs" artifact is
   generated, convert it into beads issues using `br` commands. Do not use "tasks" within
   OpenSpec—beads replaces this functionality.
    - **Issue granularity**: Create one beads issue per scenario. If a requirement has zero or
      one scenarios, create one issue for that requirement instead.
    - Use `bv` commands alongside `br` to organize beads issues.
5. **Implement and verify change-related updates**: Work on beads issues until all issues related
   to this change are closed.
    - Use `/opsx:verify` alongside `br` and `bv` commands to validate implementation.
    - Do not use `/opsx:apply` —- beads manages tasks/issues under a change instead.

#### Discovering additional tasks when working on a change

##### Tangential issues (not blocking)

When discovering issues that do not block current work (e.g., "this docstring is outdated", "this
function could be refactored"):

1. **Quick capture with beads**:
    ```bash
    br q "Refactor storage base class for better typing"
    # Created: etl-d4e5f6
    ```

2. **Add labels for categorization**: `br label add etl-d4e5f6 tech-debt discovered`.

3. **Sync so others can see it**: `br sync --flush-only`.

4. **Do NOT work on it now**—continue with the current change. Tangential issues go in the
   backlog for future prioritization.

##### Blocking issues (must fix first)

When discovering something that blocks current work (e.g., "the base class I need doesn't exist",
"there's a bug in the dependency I'm using"):

1. **Create a blocking issue**:
    ```bash
    br create "Fix fsspec credential handling" \
        --type bug \
        --priority 1 \
        --description "Discovered while implementing ingestion"
    # Created: etl-g7h8i9
    ```

2. **Add dependency**:
    ```bash
    br dep add etl-<your-current-task> etl-g7h8i9
    ```

3. **Decide how to proceed**:
    - If the blocker is small and in your domain: fix it and continue.
    - If the blocker is owned by another agent: follow the "if a conflict exists" instructions
      above.
    - If the blocker is large or outside your domain: stop and message the human.

#### Context switching

If the human asks to pause the current change and work on something else:

1. Update the coordination yaml: set `status: paused` and add a note explaining state.
2. Run `br sync --flush-only` to save any beads updates.
3. Do NOT archive—the change is incomplete.
4. Start the new change as normal.
5. When returning, read your own yaml `notes` to restore context.

#### When to stop and ask

In addition to any already mentioned occasions requiring human input (e.g., before removing or
deleting anything, as mentioned in [`AGENTS.md`](./AGENTS.md)), always stop and ask before
proceeding for:

1. **Design concerns**: A flaw, ambiguity, or potential issue with the proposal that could lead
   to:
    - Incorrect implementation.
    - Security/safety problems.
    - Significant scope creep (change targets multiple domains or is much larger than described).
    - Architectural conflict with existing code.
2. **Ambiguous requirements**: The spec or request is unclear enough that implementation would
   require guessing at intent.
3. **Technical blockers**: Something is technically impossible or requires changes outside the
   assigned scope.

### Completing a change

1. **Request sub-agent review**: After finishing "Working on a change" (all beads issues closed),
   run `/requesting-code-review` to have a subagent review the changes. Only after the subagent
   accepts the changes, present them for human review.
2. **Request human review**: Request human review on all updates for this change. Continue the
   conversation until the human accepts the updates.
3. **Sync and archive change**:
    - Use `/opsx:sync` to merge the delta PRD into the master PRD (`openspec/specs/prd.md`).
      Carefully check the master PRD afterwards to ensure the merge was done properly.
    - Use `/opsx:archive` to archive the change.
    - Reset the coordination yaml file.
    - Ask the human to do a final review and whether to make a git commit.

### Additional workflow notes

**Where the OpenSpec truth lives**:

- Current system truth: `openspec/specs/prd.md`.
- Project-level config (defaults + global context + per-artifact rules): `openspec/config.yaml`.
- OpenSpec schema to use: `openspec/schemas/*.yaml`.
- Proposed work (one folder per change): `openspec/changes/<change-name>/**`.

**Coordination rules**:

- Only ONE agent may own a file at a time, made explicit in `owned_paths` in the coordination
  yamls.
- After ANY beads update: run `br sync --flush-only`.

---

Keep this managed block so `openspec update` can refresh OpenSpec-related instructions when
needed.

<!-- OPENSPEC:END -->

---

### openspec quick reference

| Command              | Purpose                                             |
| -------------------- | --------------------------------------------------- |
| `/opsx:explore`      | Think through ideas before committing to a change.  |
| `/opsx:new`          | Start a new change.                                 |
| `/opsx:continue`     | Create the next artifact based on dependencies.     |
| `/opsx:ff`           | Fast-forward: create all planning artifacts at once.|
| `/opsx:apply`        | **Not used in OBC**—beads handles task execution.   |
| `/opsx:verify`       | Validate implementation matches artifacts.          |
| `/opsx:sync`         | Merge delta PRD into master PRD.                    |
| `/opsx:archive`      | Archive a completed change.                         |
| `/opsx:bulk-archive` | Archive multiple changes at once.                   |
| `/opsx:onboard`      | Guided tutorial through the complete workflow.      |

---

### beads-rust quick reference

**Issue Lifecycle**

| Command  | Description              | Example                              |
| -------- | ------------------------ | ------------------------------------ |
| `init`   | Initialize workspace.    | `br init`                            |
| `create` | Create issue.            | `br create "Title" -p 1 --type bug`  |
| `q`      | Quick capture (ID only). | `br q "Fix typo"`                    |
| `show`   | Show issue details.      | `br show bd-abc123`                  |
| `update` | Update issue.            | `br update bd-abc123 --priority 0`   |
| `close`  | Close issue.             | `br close bd-abc123 --reason "Done"` |
| `reopen` | Reopen closed issue.     | `br reopen bd-abc123`                |
| `delete` | Delete issue (tombstone).| `br delete bd-abc123`                |

**Querying**

| Command   | Description        | Example                                |
| --------- | ------------------ | -------------------------------------- |
| `list`    | List issues.       | `br list --status open --priority 0-1` |
| `ready`   | Actionable work.   | `br ready`                             |
| `blocked` | Blocked issues.    | `br blocked`                           |
| `search`  | Full-text search.  | `br search "authentication"`           |
| `stale`   | Stale issues.      | `br stale --days 30`                   |
| `count`   | Count with grouping.| `br count --by status`                |

**Dependencies**

| Command      | Description         | Example                          |
| ------------ | ------------------- | -------------------------------- |
| `dep add`    | Add dependency.     | `br dep add bd-child bd-parent`  |
| `dep remove` | Remove dependency.  | `br dep remove bd-child bd-parent`|
| `dep list`   | List dependencies.  | `br dep list bd-abc123`          |
| `dep tree`   | Dependency tree.    | `br dep tree bd-abc123`          |
| `dep cycles` | Find cycles.        | `br dep cycles`                  |

**Labels**

| Command        | Description            | Example                                  |
| -------------- | ---------------------- | ---------------------------------------- |
| `label add`    | Add labels.            | `br label add bd-abc123 backend urgent`  |
| `label remove` | Remove label.          | `br label remove bd-abc123 urgent`       |
| `label list`   | List issue labels.     | `br label list bd-abc123`                |
| `label list-all`| All labels in project.| `br label list-all`                      |

**Comments**

| Command         | Description    | Example                                      |
| --------------- | -------------- | -------------------------------------------- |
| `comments add`  | Add comment.   | `br comments add bd-abc123 "Found root cause"`|
| `comments list` | List comments. | `br comments list bd-abc123`                 |

**Sync & System**

| Command   | Description          | Example                 |
| --------- | -------------------- | ----------------------- |
| `sync`    | Sync DB ↔ JSONL.     | `br sync --flush-only`  |
| `doctor`  | Run diagnostics.     | `br doctor`             |
| `stats`   | Project statistics.  | `br stats`              |
| `config`  | Manage config.       | `br config --list`      |
| `upgrade` | Self-update.         | `br upgrade`            |
| `version` | Show version.        | `br version`            |

**Global Flags**

| Flag               | Description                        |
| ------------------ | ---------------------------------- |
| `--json`           | JSON output (machine-readable).    |
| `--quiet` / `-q`   | Suppress output.                   |
| `--verbose` / `-v` | Increase verbosity (-vv for debug).|
| `--no-color`       | Disable colored output.            |
| `--db <path>`      | Override database path.            |

---

### beads-viewer quick reference

Use `bv` as an AI sidecar. It is a graph-aware triage engine for Beads projects
(`.beads/beads.jsonl`). Instead of parsing JSONL or hallucinating graph traversal, use robot
flags for deterministic, dependency-aware outputs with precomputed metrics (PageRank, betweenness,
critical path, cycles, HITS, eigenvector, k-core).

**⚠️ CRITICAL: Use ONLY `--robot-*` flags. Bare `bv` launches an interactive TUI that blocks
the session.**

**`bv --robot-triage` is the single entry point**—it returns everything needed in one call:

- `quick_ref`: at-a-glance counts + top 3 picks.
- `recommendations`: ranked actionable items with scores, reasons, unblock info.
- `quick_wins`: low-effort high-impact items.
- `blockers_to_clear`: items that unblock the most downstream work.
- `project_health`: status/type/priority distributions, graph metrics.
- `commands`: copy-paste shell commands for next steps.

**`bv --robot-next` is minimal**—just the single top pick + claim command.

Use token-optimized output (TOON) for lower LLM context usage:
`bv --robot-triage --format toon`

**Planning**

| Command            | Returns                                         |
| ------------------ | ----------------------------------------------- |
| `--robot-plan`     | Parallel execution tracks with `unblocks` lists.|
| `--robot-priority` | Priority misalignment detection with confidence.|

**Graph Analysis**

| Command                                         | Returns                                                                                                                               |
| ----------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| `--robot-insights`                              | Full metrics: PageRank, betweenness, HITS (hubs/authorities), eigenvector, critical path, cycles, k-core, articulation points, slack.|
| `--robot-label-health`                          | Per-label health: `health_level` (healthy\|warning\|critical), `velocity_score`, `staleness`, `blocked_count`.                        |
| `--robot-label-flow`                            | Cross-label dependency: `flow_matrix`, `dependencies`, `bottleneck_labels`.                                                           |
| `--robot-label-attention [--attention-limit=N]` | Attention-ranked labels by: `(pagerank × staleness × block_impact) / velocity`.                                                       |

**History & Change Tracking**

| Command                               | Returns                                                                                                 |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| `--robot-history`                     | Bead-to-commit correlations: `stats`, `histories` (per-bead events/commits/milestones), `commit_index`. |
| `--robot-diff --diff-since <ref>`     | Changes since ref: new/closed/modified issues, cycles introduced/resolved.                              |

**Other Commands**

| Command                                             | Returns                                                             |
| --------------------------------------------------- | ------------------------------------------------------------------- |
| `--robot-burndown <sprint>`                         | Sprint burndown, scope changes, at-risk items.                      |
| `--robot-forecast <id\|all>`                        | ETA predictions with dependency-aware scheduling.                   |
| `--robot-alerts`                                    | Stale issues, blocking cascades, priority mismatches.               |
| `--robot-suggest`                                   | Hygiene: duplicates, missing deps, label suggestions, cycle breaks. |
| `--robot-graph [--graph-format=json\|dot\|mermaid]` | Dependency graph export.                                            |
| `--export-graph <file.html>`                        | Self-contained interactive HTML visualization.                      |

**Scoping & Filtering**

| Command                                     | Purpose                                  |
| ------------------------------------------- | ---------------------------------------- |
| `bv --robot-plan --label backend`           | Scope to label's subgraph.               |
| `bv --robot-insights --as-of HEAD~30`       | Historical point-in-time.                |
| `bv --recipe actionable --robot-plan`       | Pre-filter: ready to work (no blockers). |
| `bv --recipe high-impact --robot-triage`    | Pre-filter: top PageRank scores.         |
| `bv --robot-triage --robot-triage-by-track` | Group by parallel work streams.          |
| `bv --robot-triage --robot-triage-by-label` | Group by domain.                         |

#### Understanding robot output

**All robot JSON includes:**

- `data_hash`—fingerprint of source `beads.jsonl` (verify consistency across calls).
- `status`—per-metric state: `computed|approx|timeout|skipped` + elapsed ms.
- `as_of` / `as_of_commit`—present when using `--as-of`; contains ref and resolved SHA.

**Two-phase analysis:**

- **Phase 1 (instant):** degree, topo sort, density—always available immediately.
- **Phase 2 (async, 500ms timeout):** PageRank, betweenness, HITS, eigenvector, cycles—check
  `status` flags.

**For large graphs (>500 nodes):** Some metrics may be approximated or skipped. Always check
`status`.

#### `jq` quick reference

```bash
bv --robot-triage | jq '.quick_ref'                        # At-a-glance summary
bv --robot-triage | jq '.recommendations[0]'               # Top recommendation
bv --robot-plan | jq '.plan.summary.highest_impact'        # Best unblock target
bv --robot-insights | jq '.status'                         # Check metric readiness
bv --robot-insights | jq '.Cycles'                         # Circular deps (must fix!)
bv --robot-label-health | jq '.results.labels[] | select(.health_level == "critical")'
```

**Performance:** Phase 1 instant, Phase 2 async (500ms timeout). Prefer `--robot-plan` over
`--robot-insights` when speed matters. Results cached by data hash.

Use `bv` instead of parsing `beads.jsonl` —- it computes PageRank, critical paths, cycles, and
parallel tracks deterministically.
