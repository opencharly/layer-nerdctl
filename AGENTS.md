# AGENTS.md — layer-nerdctl

Standalone candy repo for the `layer-nerdctl` layer — nerdctl plus the rootless
containerd + buildkit + CNI stack. This repo uses the multi-directory layout: the
root `charly.yml` is the project root with `discover:` for `box/` and `candy/`,
the candy entity lives in `candy/layer-nerdctl/charly.yml`, and the disposable
bed image in `box/check-nerdctl-app/charly.yml`. The candy carries the embedded
`skill:` entity projected into the marketplace corpus as
`/charly-distros:layer-nerdctl`.

Canonical files:

- `charly.yml` — the project root: `discover:`, `defaults:`, the
  `check-nerdctl-nested:` disposable `pod:` bed (nested-pod posture), and the
  `check-nerdctl-rootless:` disposable `vm:` bed (the CONTAINERD.md §2/§4
  rootless checklist, whose runtime half needs a real systemd).
- `candy/layer-nerdctl/charly.yml` — the `layer-nerdctl:` candy entity (the
  `NERDCTL_FULL_VERSION` var, the `security:` posture, the plan steps, the
  `check:` assertions) and the `layer-nerdctl-skill:` skill entity.
- `.agents/skills/layer-nerdctl/SKILL.md` — the skill body (authored content,
  discovered from the repo root).
- `box/check-nerdctl-app/charly.yml` — the disposable bed image composing the candy.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-distros:layer-nerdctl` — the owning skill. The containerd engine
  (driven through the nerdctl CLI), the `layer-nerdctl` candy, the nested-pod
  posture, and `engine: containerd` deploys.
  Load before editing or troubleshooting the layer.
- `/charly-internals:plugin-nerdctl` — the out-of-tree engine plugin that serves
  `engine: containerd`.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs incl. `mkdir:`/`write:`/`check:`, per-distro `distro:` arms,
  package/repo sections, and service declarations). Load before editing any
  entity field or plan step.

## Build / validate / test

- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no
  per-repo candy gate** and ships no per-repo workflow file — the CalVer tag and
  `CHANGELOG/` entry are written by the org-wide `tag-on-merge` dispatcher on merge.
- The candy's `plan:` `check:` steps are the functional evidence; they must stay
  valid on every distro arm they run on. Scope a distro-specific check in the
  command itself — the check runner does not honour runner-level
  `exclude-distro` fields.
- The R10 for the nested-pod posture is the `check-nerdctl-nested:` disposable
  `pod:` bed in the root `charly.yml` — a FRESH deploy asserting `/dev/fuse`,
  `/dev/net/tun`, the full `CapEff` mask, the userns uid map, and `nerdctl` on
  PATH. It fails without the `security:` block.
- The R10 for the rootless checklist is `check-nerdctl-rootless:` — a disposable
  `vm:` bed, because the checklist's runtime half (`systemctl --user`, linger,
  the units) cannot be proven in a supervisord pod.

## Modify this repo

- Edit the `layer-nerdctl:` candy entity in `candy/layer-nerdctl/charly.yml` AND
  the skill body in `.agents/skills/layer-nerdctl/SKILL.md` together. The
  `layer-nerdctl-skill:` node carries ROUTING only; the file is the source of
  record for the skill's prose, so a behaviour change not mirrored there leaves
  the corpus stale.
- The pinned `NERDCTL_FULL_VERSION` and its sha256 verification are the contract;
  the native-package distro arms (arch/alpine) short-circuit the tarball step —
  preserve that split.
- The `security:` posture is userns-scoped (`cap_add: ALL`, `/dev/fuse`,
  `/dev/net/tun`, `unmask=/proc/*`); the inner engine's userns-root is NOT host
  root. Keep the bed's `check:` assertions aligned with it.
- New behaviour claims belong in the `plan:` as an observable `check:` step, and
  in the skill body.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge
  on PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the
  PR body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
