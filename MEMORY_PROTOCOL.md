# Memory Protocol

Agents do not remember between sessions. A project's memory folder is how work survives from one session, one agent, and one day to the next. Every agent working on the project reads it before acting and writes to it as it works, so that an agent starting tomorrow knows what we are trying to achieve, where things stand, and what happened.

This protocol is governed by `AGENTS.md`. It does not grant any permission that file withholds.

---

## Layout

Each project has one memory folder at its root:

```
<project>/memory/
├── GOALS.md        where we are going: outcomes we are working toward
├── INDEX.md        where we are now: current state, corrected in place
└── YYYY-MM-DD.md   what happened that day: append-only, one file per day
```

Three files, three jobs. Do not let one absorb another:

| File | Answers | Changes | Edited how |
| --- | --- | --- | --- |
| `GOALS.md` | What are we trying to achieve, and how will we know? | Rarely, only by the user | In place |
| `INDEX.md` | What is true right now? | Whenever the current state changes | In place, kept short |
| `YYYY-MM-DD.md` | What happened, decided, and broke that day? | Continuously | Append-only |

The memory folder holds project knowledge, not code. Keep it out of public repositories (add `memory/` to `.gitignore`) unless the user explicitly decides otherwise.

---

## Reading

### At the start of every session

1. Read `GOALS.md`.
2. Read `INDEX.md`.
3. Read the two or three most recent day files.

### When you do not know something

Before asking the user, search the memory folder for the topic (names, identifiers, keywords) across every day file. When entries disagree, the newest one wins, unless an older entry records a decision the user has not reversed.

### Memory is a claim, not a fact

A memory entry records what was true when it was written. Before acting on it (especially file paths, versions, identifiers, ownership, or the state of a live system), verify it against the code or the system. If it turns out to be wrong, record the correction in today's file and fix `INDEX.md`.

---

## Writing

### Who writes

Every agent working on the project. There is no single owner of memory: each agent records its own work.

### When to write

- At the end of any substantial task.
- Before handing work to another agent or session, and before your context is compacted or reset.
- At the end of the day.
- **Immediately** when the user makes a decision, rejects an option, parks something, or assigns ownership. Do not batch these until later; they are the entries most often lost.

### Rules for every write

- **Your block, your words.** Append your own block to today's file. Never edit or delete another agent's block. If an earlier entry is wrong, say so in your own block and correct `INDEX.md`.
- **Quote the user** for decisions, rejections, and parked items. Their exact words carry intent that a paraphrase loses.
- **Evidence over narrative.** Commits, merge requests, test counts, command output, measured numbers. "Achieved" means verified, not attempted.
- **Name owners.** Every open item says who owns it: the user, a named person, or an agent.
- **No secrets.** Never write tokens, passwords, keys, or credentials. Write where they live instead.
- **Link, don't copy.** `INDEX.md` points at the day a decision was made; it does not restate the whole history.

---

## `GOALS.md`

Goals are the user's. Agents may propose a goal under **Proposed**; only the user promotes it to **Active**, changes it, or drops it.

```markdown
# Goals

## Active

### <Goal name>
- **Outcome:** what will be true when this is achieved
- **Why:** why it matters
- **Done when:** a concrete, checkable condition
- **Owner:** <who>
- **Since:** YYYY-MM-DD

## Proposed

### <Goal name>
- **Outcome:** ...
- **Proposed by:** <agent/session>, YYYY-MM-DD, because ...

## Achieved

### <Goal name> (achieved YYYY-MM-DD)
- **Evidence:** what proved the "done when" condition

## Dropped

### <Goal name> (dropped YYYY-MM-DD)
- **Why:** "<user's words>"
```

---

## `INDEX.md`

The current state of the project and nothing else. Corrected in place: when an item finishes, it leaves the index; its history stays in the day files. Keep it to roughly 150 lines. If it grows past that, it is holding history.

```markdown
# Index

_Last updated: YYYY-MM-DD by <agent/session>_

## Active
- **<Work item>** (Goal: <goal>): <current state>. Next: <step>. Owner: <who>. See YYYY-MM-DD.

## Parked
- **<Item>**: "<user's words>" (YYYY-MM-DD). Do not raise unprompted. If revived, start from: <pointer>.

## Standing decisions
- **<Decision>**: <reason>. ("<user's words>", YYYY-MM-DD)

## Rejected (do not propose again)
- **<Option>**: rejected because <reason>. ("<user's words>", YYYY-MM-DD)

## Traps
- **<Pitfall>**: <what goes wrong and the safe sequence>.

## Ownership
- **<Area or item>**: <owner>
```

---

## `YYYY-MM-DD.md`

One file per day. Many agents, many blocks, append-only. The first agent of the day creates the file.

```markdown
# YYYY-MM-DD

## HH:MM · <agent/session> · Goal: <goal from GOALS.md, or "none">

**Set out to:** what this session meant to do
**Done:** what was actually done (evidence: commits, MRs, tests, output)
**Achieved:** the verified outcome, not just that something ran
**Decisions:** <decision>, because <reason>. ("<user's words>")
**Rejected:** <option>, because <reason>
**Incidents:** what broke, impact, fix
**Open:** <item> (owner: <who>)
**Next:** what the next agent should pick up first

---

## End of day

- **Moved:** which goals advanced, and how
- **Tomorrow:** what is on whose plate
- **Index updated:** yes / no
```

Leave out sections that have nothing to say. Do not leave out **Open** or **Next** if work is unfinished.

---

## Checklist before ending a session

- [ ] Today's file has my block, with evidence.
- [ ] Every decision, rejection, and parked item from this session is recorded with the user's words.
- [ ] `INDEX.md` reflects the current state; finished items are removed.
- [ ] Every open item has an owner.
- [ ] No secrets were written.
