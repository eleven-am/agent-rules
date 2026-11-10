# agent rules

Rules for AI coding agents, independent of which agent you use. Three files:

- **[`AGENTS.md`](AGENTS.md)**: the execution policy. How an agent is allowed to act: plan approval before any change, Git locked by default, no code comments except actionable TODOs, mandatory tests and builds, packages over reinvention, and no AI attribution in commits.
- **[`CODING_PROFILE.md`](CODING_PROFILE.md)**: the engineering standard. What good work looks like: one owner per fact and behavior, transports as thin adapters over operations, explicit failure over hidden fallback, hard cutoffs over legacy paths, honest tests, and frontend architecture where applicable.
- **[`MEMORY_PROTOCOL.md`](MEMORY_PROTOCOL.md)**: how work survives between sessions. Each project keeps a `memory/` folder with `GOALS.md` (where we are going), `INDEX.md` (where we are now), and one file per day (what happened). Every agent reads it before acting and writes to it as it works.

`AGENTS.md` takes precedence where the files touch. Project-specific instructions add local context on top; they never weaken these rules.

## Usage

Copy the three files to wherever your agent reads its global instructions, or to the root of a project. Most coding agents read `AGENTS.md` at the repository root; if yours expects a different file name, point that file at `AGENTS.md`.

In each project:

```bash
mkdir -p memory
echo "memory/" >> .gitignore
```

Then add a short project section, or a separate file, for local architecture, scope, and source-of-truth documents.

The approval phrases used by `AGENTS.md`:

| Phrase | Unlocks |
| --- | --- |
| `PLAN APPROVED — PROCEED` | Implementing the approved plan |
| `ENABLE GIT OPERATIONS NOW` | Git commands for the current session (pushing still needs a separate request) |
| `ACCESS THE OLD FOLDER NOW` | Reading paths containing `old/` |

## License

[MIT](LICENSE)
