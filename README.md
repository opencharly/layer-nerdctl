# layer-nerdctl

nerdctl (the containerd CLI) plus the full rootless containerd + buildkit + CNI
stack, as a standalone OpenCharly layer repo.

Composed by a box that wants the opt-in `engine: containerd` engine. On
Arch/CachyOS/Omarchy and Alpine the candy uses native distro packages; on Fedora,
Debian, and Ubuntu it installs the pinned, **fatally** sha256-verified
`nerdctl-full` tarball (which supplies nerdctl + containerd +
`containerd-fuse-overlayfs-grpc` + CNI + buildkit + rootlesskit in one archive).
It writes `/etc/nerdctl/nerdctl.toml` (`namespace = "default"`), the `charly` CNI
network conflist at `/etc/cni/net.d/charly.conflist`, and the rootless
containerd/buildkit user units.

## What it provides

| Property | Value |
|---|---|
| Layer / candy | `layer-nerdctl` |
| Pinned version | `NERDCTL_FULL_VERSION` `2.1.2`, sha256-verified fatally |
| Config | `/etc/nerdctl/nerdctl.toml` (`namespace = "default"`), `/etc/cni/net.d/charly.conflist`, CNI plugins at `/opt/cni/bin` |
| Rootless units | `~/.config/systemd/user/containerd.service`, `~/.config/systemd/user/buildkit.service` |
| Rootless checklist | uidmap helpers, `/etc/subuid` + `/etc/subgid` (`user:100000:65536`), `user@.service.d/delegate.conf`, linger, `/dev/fuse`, user D-Bus, slirp4netns (CONTAINERD.md §2, §4) |
| Nested-pod posture | userns-scoped `cap_add: ALL`, `/dev/fuse`, `/dev/net/tun`, `unmask=/proc/*` |
| Service / port | none |

The `engine: containerd` word is served by `opencharly/plugin-nerdctl`
(out-of-process) or compiled in; every engine op delegates to
`container.InvokeEngineOp("containerd", …)`. `containerd` is the engine word;
`nerdctl` is the CLI binary that drives it (the spec-owned `#EngineBinaries` map
holds `containerd → nerdctl`).

## Rootless networking

`slirp4netns` is the default and is pinned in the containerd unit. `pasta` and
`bypass4netns` are documented options, not recommendations: on rootlesskit 3.3.0
pasta starts (`rootlesskit --net=pasta true`, rc 0) but the deployment's
networking properties are measured on slirp4netns, and bypass4netns is refused by
that rootlesskit (`unknown network mode` / `unknown port driver`). Publishing a
port below 1024 is governed by the host's
`net.ipv4.ip_unprivileged_port_start`; this layer never sets it — see the skill
for when to do it deliberately.

## How to use it

This repo uses the multi-directory layout: the root `charly.yml` is the project
root with `discover:` for `box/` and `candy/`, and the candy lives in
`candy/layer-nerdctl/charly.yml`. Compose it as a nested `candy:` list inside a
named box body:

```yaml
my-nested-box:
  candy:
    base: fedora
    candy:
      - '@github.com/opencharly/layer-nerdctl:v2026.266.1833'
```

## Layout

- `charly.yml` — the project root: `discover:`, `defaults:`, the
  `check-nerdctl-nested:` disposable `pod:` bed (nested-pod posture), and the
  `check-nerdctl-rootless:` disposable VM bed (the §4 rootless checklist).
- `candy/layer-nerdctl/charly.yml` — the `layer-nerdctl:` candy entity, its
  `security:` posture, the plan steps/checks, and the `layer-nerdctl-skill:`
  skill entity.
- `.agents/skills/layer-nerdctl/SKILL.md` — the skill body.
- `box/check-nerdctl-app/charly.yml` — the disposable bed image composing the candy.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-distros:layer-nerdctl` — the containerd engine (driven
  through the nerdctl CLI), the `layer-nerdctl` candy, the rootless checklist,
  and `engine: containerd` deploys.
- `/charly-internals:plugin-nerdctl` — the out-of-tree engine plugin.
- `/charly-image:layer` — candy authoring reference.
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
