
These instructions apply to AI agents working in this repository.

## Project overview

See `README.md` and `openspec/specs/prd.md` for a more detailed overview of *what* this project
is about and its requirements. This `AGENTS.md` doc is for understanding *how* to work on this
project.

---
## Practices to follow

### Rules

You MUST adhere to these rules—these are non-negotiable. This is for the development of robust
software that would otherwise be compromised.

1. THE FUNDAMENTAL OVERRIDE PREROGATIVE: If in a conversation I tell you to do something, even if
   it goes against what follows below, YOU MUST LISTEN TO ME.
2. NEVER delete any file or directory, git commit any changes, nor git push any changes, unless I
   explicitly grant you permission to do so in this session. If you think *anything* should be
   removed or deleted, git committed, or git pushed, you MUST stop and ask. Again, I must
   explicitly give permission for a delete, commit, or push in order for you to do so. Treat
   "never delete files without permission", "never git commit without permission", and "never git
   push without permission" as hard constraints.
3. Brevity is the soul of wit. Generally be succinct unless I specify otherwise, or you really
   really think it would help an explanation for my understanding.
4. Write google-style, production-quality code. Prefer efficiency and minimality over
   readability. Show off your strong coding skills and knowledge of powerful patterns,
   particularly with jax, pytorch, einops, and numpy, rather than defaulting to novice code.
   Ensure you generally optimize for space and time complexity: e.g. when applicable, use
   vectorization over for loops, hash maps over repeated linear scans (e.g. membership checks),
   caching over repeating pre-computations, etc.

---

### Shell and Python best practices

1. Use `zsh` over `bash` by default.
2. Use `uv` to run things in this project's environment and to handle package management (e.g.
   `uv run python...`, `uv run ruff ...`, `uv run pytest ...`).
3. In the rare cases where I do ask you to make a commit, use conventional commit messages.
4. Use einops extensively, as default for tensor and array operations when applicable.
5. Use jaxtyping extensively, as default whenever declaring/defining tensors and arrays.
6. Use beartype extensively, as default for decorating any function or class that operates on
   tensors or arrays (with jaxtyped via `@jaxtyped(typechecker=beartype)`), and for all "public"
   functions and classes users call directly.
7. Use treescope as default for printing and viz, particularly in notebooks.
8. Adhere to google python style conventions by default, and look at `pyproject.toml` and
   `.pre-commit-config.yaml` to see potential additional conventions to adhere to.
9. Always add or update tests for any non-trivial logic change; prefer fast unit tests.

---

<!-- OPENSPEC:START -->

## Agent workflow: "openspec + beads + coordination"

In [AGENT_WORKFLOW.md](./AGENT_WORKFLOW.md) I describe a general "openspec + beads + coordination"
(OBS) workflow to follow when working on this project.

Cases in which you don't have to use the OBS workflow are:
1. If I have *not* assigned you a name at the start of the conversation and you are *not* in a
   git worktree of the main git project.
    - You can use `git rev-parse --git-common-dir` to check: if the returned output is just
      `.git`, you are in the main, common dir, else you are in a worktree.
2. When I'm asking you general questions that do not entail you writing or editing code.
3. When I'm asking you to do small, trivial, one-off code changes. e.g.: "Can you update this
   function's docstring?", "Can you quickly write this (small) function?", "Can you quickly write
   some scratch code that...", "Can you add a cell to this notebook that...", etc.

*When I have: (1) assigned you a name and (2) you are in a git worktree and (3) a request
involves planning, proposals, specs, architecture shifts, or non-trivial performance/security
changes, read and use the OBS workflow ([AGENT_WORKFLOW.md](./AGENT_WORKFLOW.md)).*

---

## References

### `agent_assets/`

`agent_assets/` is a key directory in this project that contains assets for you to reference
while working on this project. It contains information on:
- <example_topic1>
- <example_topic2>

When I ask you a question about any of those bullet points, unless you're very confident that you
already know the answer, the first place you should look is `agent_assets/`.

...
