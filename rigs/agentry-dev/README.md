# agentry-dev

A four-seat OpenRig team (lead, builder, code-reviewer, QA) that builds
Agentry itself: the rigs, workflows, ADRs and culture the other departments
run on. A person brings the goal; the team delivers one pull request at a
time. Install it once as `agentry-dev` and launch it in the clone you're
working on.

| Seat                     | Runtime     | Does                                                                                |
| ------------------------ | ----------- | ----------------------------------------------------------------------------------- |
| `orch-lead@agentry-dev`  | codex       | Takes your goal, scopes it, hands it to the builder, keeps the PR description true. |
| `dev-build@agentry-dev`  | claude-code | Makes the change on its own branch and proves it works.                             |
| `dev-review@agentry-dev` | codex       | Reviews the handed-off commit against the ADRs and the files.                       |
| `dev-qa@agentry-dev`     | claude-code | Runs the checks the change calls for and reports what each proved.                  |

`CULTURE.md` holds the team's values. Each seat's role is in
`agents/agentry-dev/guidance/`.

## Where it comes from

Adapted from
[`pilotariak-dev`](https://github.com/pilotariak/fabrika/tree/551bff7daf890b3e78bf4e418681593d997a5164/rigs/pilotariak-dev):
same topology, presets and lean design, with the Pilotariak-specific rules
(repo map, `frontis` hub, design tokens, the `fabrika` pack) replaced by
Agentry's (ADRs first, rig and workflow validation, guardrails that only an
accepted ADR can loosen).

## Why two rigs

`factory` is autonomous and works on a **product**. `agentry-dev` is
human-steered and works on **Agentry's own rules**. Keeping them apart means
the loop that runs without a human never edits its own guardrails.

## Before you start

- OpenRig 0.6.6 or later (`rig --version`), Node 22 or 24, and tmux.
- Claude Code and/or Codex installed and logged in — `recommended` uses both;
  `all-claude`/`all-codex` need only one.

Keep OpenRig's files out of your commits. From the root of your clone:

```bash
printf '%s\n' CLAUDE.local.md AGENTS.md /.openrig/ >> .git/info/exclude
```

## Run it

From the root of the clone you want to work on:

```bash
rig bundle install <path-or-url-to>/rigs/agentry-dev --preset recommended --target ~/rigs/agentry-dev --cwd . --plan
rig bundle install <path-or-url-to>/rigs/agentry-dev --preset recommended --target ~/rigs/agentry-dev --cwd .
```

Send the lead a goal:

```bash
rig send orch-lead@agentry-dev "Please take issue #<number>: <one line on what you want>"
```

Stop the team with `rig down agentry-dev`.

## Permissions

No `permission_policy` is set, so every seat launches under its runtime's
normal prompts: Claude Code seats ask before file edits, and Codex seats
(`orch-lead` and `dev-review` in `recommended`) run with `workspace-write` —
**no network, including the local OpenRig daemon**, unless you opt a seat into
a networked config profile (OpenRig's `docs/reference/getting-started.md`).
Use `all-claude` if you don't want to set that up.

Don't add `permission_policy: builtin:yolo` here without an accepted ADR: this
team edits the guardrails the other rigs live by.

## Limits

- **A starter, not a process.** One pull request at a time; merging and
  releases stay with you.
- **No mission/slice ledger.** Re-brief the lead if a seat restarts
  mid-task.
- **Nothing publishes on its own.** The team asks for your word before it
  pushes, opens a pull request, or comments.
