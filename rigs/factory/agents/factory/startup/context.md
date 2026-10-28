# Start (every seat on this rig)

1. Run `rig whoami --json`. It names your seat and your peers' sessions. Use
   those names; don't guess addresses. During launch a peer may not be listed
   yet — if one of the seven seats is missing, check `rig ps --nodes` again
   shortly before telling anyone it's missing.
2. Find the product repository you work in: `git rev-parse --show-toplevel`.
   If that fails, or your working directory is this rig's own folder, the rig
   was launched without `--cwd` — stop and say so.
3. Read that repo's own maps before anyone changes anything: its `AGENTS.md`
   if it has one, its `README`, and `make help` (or the equivalent) for the
   commands it exposes. Those specs and docs are the product corpus.
4. If `rig context list` shows an Agentry `handbook` pack, load it:
   `rig context profile handbook --situation fresh`. Read it in addition to
   step 3's repo-local maps, not instead of them.

Last, submit your startup proof: run the `rig startup-proof submit` command
from the "OpenRig startup orientation challenge" in your startup text, exactly
as given.

Then wait. The `factory` workflow routes work to you as a packet; the
orchestrator only hears about exceptions. An idle seat is fine — idle roles
are capacity, not a reason to invent work. The dogfood seat is the one
exception: it starts using the shipped product once the human or the
orchestrator kicks it off.
