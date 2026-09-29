# AGENTS.md — layer-nerdctl

Standalone candy repo for the `layer-nerdctl` layer — nerdctl plus the rootless
containerd + buildkit + CNI stack. This repo uses the multi-directory layout: the
root `charly.yml` is the project root with `discover:` for `box/` and `candy/`,
the candy entity lives in `candy/layer-nerdctl/charly.yml`, and the disposable
bed image in `box/check-nerdctl-app/charly.yml`. The candy carries the embedded
`skill:` entity projected into the marketplace corpus as
`/charly-distros:layer-nerdctl`.

Canonical files:

- `charly.yml` — the project root: `discover:`, `defaults:`, and the
  `check-nerdctl-nested:` disposable `pod:` R10 bed.
- `candy/layer-nerdctl/charly.yml` — the `layer-nerdctl:` candy entity (the
  `NERDCTL_FULL_VERSION` var, the `security:` posture, the plan steps, the
  `check:` assertions) and the `layer-nerdctl-skill:` skill entity.
- `box/check-nerdctl-app/charly.yml` — the disposable bed image composing the candy.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-distros:layer-nerdctl` — the owning skill. The nerdctl engine, the
  `layer-nerdctl` candy, the nested-pod posture, and `engine: nerdctl` deploys.
  Load before editing or troubleshooting the layer.
- `/charly-internals:plugin-nerdctl` — the out-of-tree engine plugin that serves
  `engine: nerdctl`.
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

## Modify this repo

- Edit the `layer-nerdctl:` candy entity AND the `layer-nerdctl-skill:` skill
  entity in `candy/layer-nerdctl/charly.yml` together. The skill is the projected
  usage source, so a behaviour change not mirrored in the skill leaves the corpus
  stale.
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
