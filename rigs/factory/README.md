# factory

Agentry's engineering department: a seven-seat OpenRig rig that runs a
recursive improvement loop on a product repository. The `factory` workflow
moves one slice through **plan → implement → check → review → release prep**,
then holds the ship decision at a human gate. A dogfood seat uses the
**shipped** product out-of-band and its recorded findings become the next
plan.

| Seat                        | Runtime     | Does                                                                  |
| --------------------------- | ----------- | --------------------------------------------------------------------- |
| `plan-planner@factory`      | claude-code | Turns the product corpus or the last dogfood findings into one slice. |
| `build-implementer@factory` | claude-code | Builds the slice on its own branch and records proof.                 |
| `check-qa@factory`          | codex       | Checks the artifact; a failing check loops back to build.             |
| `review-reviewer@factory`   | codex       | Independent review on the other runtime; a failing review loops back. |
| `dogfood-tester@factory`    | codex       | Uses the shipped product and records findings for the next plan.      |
| `release-manager@factory`   | claude-code | Prepares notes, docs and `proof/PROOF.md`; stops at the human gate.   |
| `orch-lead@factory`         | claude-code | Exception target only — the workflow engine routes the loop.          |

`CULTURE.md` holds the values. Each seat's role is in
`agents/factory/guidance/`, and the loop itself is `workflow/factory.yaml`.

## Where it comes from

Forked from OpenRig's `factory-rsi` reference
([openrig-world `rigs/launch/factory-rsi`](https://github.com/mvschwarz/openrig-world/tree/52468eea83f9b875a156e60f04c77c28cc7034aa/rigs/launch/factory-rsi),
Apache-2.0, commit `52468eea83f9b875a156e60f04c77c28cc7034aa`) and its
built-in `factory-rsi` workflow, packaged the way
[`pilotariak-dev`](https://github.com/pilotariak/fabrika/tree/551bff7daf890b3e78bf4e418681593d997a5164/rigs/pilotariak-dev)
is. Differences from upstream:

- **Self-contained.** Upstream seats point at `../../../agents/...` and import
  OpenRig's `shared` skills and `openrig-core` plugin from outside the
  bundle. Here one local agent (`agents/factory`) carries seven profiles, and
  each role's method is written into its guidance file.
- **Own workflow.** `workflow/factory.yaml` is upstream's `factory-rsi`
  workflow with the rig renamed to `factory` and `skill_refs` reduced to
  `openrig-user`. The steps, gates, and `max_hops: 20` are unchanged.
- **Agentry rules.** DCO sign-off, no `Co-authored-by`, and "the human is the
  board" (ADR-0001).
- **Presets.** `recommended` keeps upstream's runtime split; `all-claude` and
  `all-codex` need only one provider.

## Before you start

- OpenRig 0.6.6 or later (`rig --version`), Node 22 or 24, and tmux.
- Claude Code and/or Codex installed and logged in.
- A clone of the product repository the factory should improve. Its specs,
  `README` and `AGENTS.md` are the planner's corpus on cycle 1.

Keep OpenRig's files out of that repository's commits:

```bash
printf '%s\n' CLAUDE.local.md AGENTS.md /.openrig/ /proof/ >> .git/info/exclude
```

## Run it

From the root of the product clone:

```bash
rig bundle install <path-or-url-to>/rigs/factory --preset recommended --target ~/rigs/factory --cwd . --plan
rig bundle install <path-or-url-to>/rigs/factory --preset recommended --target ~/rigs/factory --cwd .
```

Or, from a clone of this repository, preview the spec directly:

```bash
rig up rigs/factory/rig.yaml --cwd <product-clone> --plan
```

Start a cycle once the seven seats show `ORIENTED` in `rig ps --nodes`. The
workflow path must be absolute (or relative to where you run the command),
since you're usually in the product clone, not in this repository:

```bash
AGENTRY=<path-to-your-agentry-clone>
rig workflow validate "$AGENTRY/rigs/factory/workflow/factory.yaml"
rig workflow instantiate "$AGENTRY/rigs/factory/workflow/factory.yaml" \
  --root-objective "<what this cycle should improve>" \
  --created-by orch-lead@factory
rig workflow status
```

Install it as `factory`. `rig workflow instantiate --rig <name>` overrides the
spec's `target.rig`, but whether the fully qualified `preferred_targets`
(`<seat>@factory`) follow it is untested. Stop the rig with
`rig down factory`.

## Permissions

No `permission_policy` is set, so every seat launches under its runtime's
normal prompts: Claude Code seats ask before file edits, and Codex seats run
with `workspace-write` — **no network, including the local OpenRig daemon**,
unless you opt a seat into a networked config profile (OpenRig's
`docs/reference/getting-started.md`). In `recommended`, `check-qa`,
`review-reviewer` and `dogfood-tester` are Codex seats: they must reach the
daemon to close their workflow step, and dogfood needs network to use a
shipped product. Give those seats a networked profile, or use `all-claude`,
before you rely on the loop.

The faster, prompt-free posture (`permission_policy: builtin:yolo` and
`non_interruptive: true`) lets every seat change files, install packages and
use the network without asking. Don't enable it until the loop is proven
(ADR-0001); the human gate on release still holds either way.

## Limits

- **Dogfood is not automatic yet.** Upstream marks the continuous out-of-band
  trigger as "refined in a later release". Start the dogfood seat yourself
  (`rig send dogfood-tester@factory "..."`) or schedule it with
  `rig watchdog`, and start the next cycle once its findings are recorded.
- **One cycle per instance.** Each `rig workflow instantiate` runs one slice;
  closing the loop again is a new instance.
- **Nothing ships on its own.** `release_signoff` waits at `human@kernel`;
  publishing, tagging and merging stay with you.
- **Host dependency.** Seats load OpenRig's `openrig-core` plugin
  (`openrig-home:plugins/openrig-core`), which resolves on the installing
  host; it is not vendored here.
- **Every guidance file here can be stale.** The product's own code and CI
  decide.
