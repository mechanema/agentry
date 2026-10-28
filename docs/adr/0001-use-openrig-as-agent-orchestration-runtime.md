---
adr: 0001
status: ✅ Accepted
deciders: Nicolas Lamirault
consulted:
informed:
date: 2026-10-27
spdx-license: Apache-2.0
---

# ADR-0001: Use OpenRig as the Agent Orchestration Runtime

## Context

Agentry is a company staffed only by AI agents. A human acts as the board: they set goals and approve anything that is
published, paid for, or legally binding. Everything else (planning, building, checking, reviewing, releasing, dogfooding,
research, documentation) is done by agents.

That requires a runtime able to:

- Run several long-lived agents ("seats") side by side, each with a clear role, and group them into departments.
- Mix agent runtimes (Claude Code, Codex) so that review and QA do not share the builder's blind spots.
- Describe a team declaratively, version it in git, and reinstall it reproducibly on any machine.
- Route work between seats through recorded state (a queue, a workflow engine), not through chat history, so that work
  survives restarts and compactions.
- Bound autonomous loops (hop limits, exception routing) and keep human gates on publishing.
- Share company-wide context (values, repo map, how a change is proven) with every seat at startup.

Three existing references informed this decision:

- [`openrig-world/rigs/launch/factory-rsi`](https://github.com/mvschwarz/openrig-world/tree/main/rigs/launch/factory-rsi):
  a seven-seat recursive self-improvement loop (plan → build → check → review → release, plus out-of-band dogfood).
- [`pilotariak/fabrika/rigs/pilotariak-dev`](https://github.com/pilotariak/fabrika/tree/551bff7daf890b3e78bf4e418681593d997a5164/rigs/pilotariak-dev):
  a lean four-seat OpenRig team with a `CULTURE.md`, a context pack, and a human publishing gate.
- [`scion-frontiers/agent-team/templates`](https://github.com/scion-frontiers/agent-team/tree/main/templates): a library of
  role templates (coordinator, eng-manager, researcher, security-auditor, doc-writer, …) for the Scion framework.

## Considered Options

1. **OpenRig**: declarative rigs (`rig.yaml`) of pods and seats, multi-runtime, with bundles, context packs, workflows,
   and a queue.
2. **Scion**: the orchestration framework behind `scion-frontiers/agent-team`, with a rich library of role templates.
3. **Claude Code native subagents / agent teams**: subagents and workflows inside a single Claude Code session.
4. **Custom orchestration**: build an in-house orchestrator on top of the Claude Agent SDK and the Codex CLI.

## Pros and Cons

### 1. OpenRig

**Pros:**

- ✅ Declarative team topology (`pods`, `members`, `edges` such as `delegates_to` and `can_observe`), versioned in git.
- ✅ Multi-runtime by design: each seat chooses `claude-code` or `codex`, and `configurations.yaml` presets
  (`recommended`, `all-claude`, `all-codex`) adapt the team to the providers that are logged in.
- ✅ Workflow engine that routes the inner loop and enforces `max_hops`, with exceptions escalated to an orchestrator seat
  first. `factory-rsi` is a working reference for an autonomous improvement loop.
- ✅ Shared culture (`culture_file`) and context packs (`rig context add/profile`) give every seat the same values and
  maps without copying them into each prompt.
- ✅ Startup proofs (`rig ps --nodes` shows `ORIENTED`) and a queue for handoffs that survive restarts.
- ✅ Bundles (`rig bundle install … --plan`) make a department installable and previewable on any host.
- ✅ Prior hands-on experience: `pilotariak-dev` is already authored and running against it.

**Cons:**

- ❌ Young project (minimum CLI/daemon `0.6.6`); formats have already migrated once (`world-bundle.yaml` is a preserved
  copy of the older format).
- ❌ Local-first: depends on tmux and Node 22/24 on the host; no managed cloud execution.
- ❌ Some reference rigs depend on resources outside the bundle (`openrig-home:plugins/openrig-core`, shared `agents/`
  paths), so they must be vendored before reuse.
- ❌ Routing work between rigs (departments) is not yet validated for our use.

### 2. Scion

**Pros:**

- ✅ Large, ready-made catalog of roles beyond engineering (coordinator, researcher, doc-writer, security-auditor,
  release-notes, web-builder).
- ✅ Skills are shared by URI (`gh://scion-frontiers/agent-team/...`), so roles stay small.

**Cons:**

- ❌ No hands-on experience with it yet, and no existing Agentry or Pilotariak team runs on it.
- ❌ Its role definitions (`scion-agent.yaml`, `system-prompt.md`, `agents.md`) do not fit OpenRig's model, so mixing the
  two would mean running two orchestration stacks.

### 3. Claude Code native subagents / agent teams

**Pros:**

- ✅ No extra runtime to install; already part of the daily toolchain.
- ✅ Good fit for short, bounded fan-out tasks inside one session.

**Cons:**

- ❌ Single runtime: no cross-runtime diversity between builder and reviewer.
- ❌ Teams live inside a session, with no declarative, reinstallable team topology and no long-lived departments.

### 4. Custom orchestration

**Pros:**

- ✅ Full control over routing, state, budgets, and observability.

**Cons:**

- ❌ Large build and maintenance cost before the company ships anything of its own.
- ❌ Reinvents what OpenRig already provides (seats, queue, workflows, bundles, context packs).

## Decision

We will use *OpenRig* as Agentry's agent orchestration runtime. It is the only option that combines declarative,
versioned teams, multi-runtime seats, a bounded workflow engine, shared culture and context packs, and a human gate on
publishing. We already have working experience with it through `pilotariak-dev`.

This decision implies:

- Each department is a separate rig (for example `exec`, `factory`, `research`, `growth`, `trust`), not one large rig.
  Agentry's own repositories (rigs, ADRs, culture, handbook) are maintained by a separate human-steered rig,
  `agentry-dev`, derived from `pilotariak-dev`.
- Company values live in a shared `CULTURE.md`, and company knowledge lives in a context pack that every seat loads at
  startup.
- The engineering rig starts as a fork of `factory-rsi`, with its agents vendored locally as `pilotariak-dev` does.
- Roles from `scion-frontiers/agent-team` are reused as content and ported to OpenRig `agent.yaml` profiles. Scion
  itself is not run.
- The code-review seat always runs on a different runtime from the builder. Other checking seats (QA, dogfood) do so
  where the rig's preset allows it.
- No seat pushes, tags, publishes, or spends money without the human's approval. `permission_policy: builtin:yolo`
  stays off until a rig's loop has been proven.

## Consequences

### Positive

- ✅ Teams are code: reviewed, versioned, and reinstallable with `rig bundle install`.
- ✅ Cross-runtime review and bounded loops come built in, which limits runaway autonomy.
- ✅ Work hands off through recorded state (queue, workflow), so departments survive restarts and compactions.
- ✅ Existing assets (`factory-rsi`, `pilotariak-dev`, the `fabrika` context-pack pattern) can be reused directly.

### Negative

- ❌ Agentry is coupled to a young tool and will absorb its format migrations and breaking changes.
- ❌ Every operator host needs OpenRig, tmux, Node, and logged-in Claude Code and/or Codex runtimes.
- ❌ Porting Scion roles to OpenRig profiles is manual work.
- ❌ Routing work between rigs must be validated before the multi-department phase.

### Neutral

- ↔️ Pin `min_cli_version` / `min_daemon_version` in every `bundle.yaml` and track OpenRig upgrades deliberately.
- ↔️ Revisit this ADR if cross-rig coordination proves insufficient or a managed cloud runtime becomes necessary.

## References

- [OpenRig](https://github.com/mvschwarz/openrig)
- [openrig-world: factory-rsi](https://github.com/mvschwarz/openrig-world/tree/main/rigs/launch/factory-rsi)
- [pilotariak/fabrika: pilotariak-dev](https://github.com/pilotariak/fabrika/tree/551bff7daf890b3e78bf4e418681593d997a5164/rigs/pilotariak-dev)
- [scion-frontiers/agent-team: templates](https://github.com/scion-frontiers/agent-team/tree/main/templates)
