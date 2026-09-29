# layer-nerdctl

nerdctl (the containerd CLI) plus the full rootless containerd + buildkit + CNI
stack, as a standalone OpenCharly layer repo.

Composed by a box that wants the opt-in `engine: nerdctl` engine. On
Arch/CachyOS/Omarchy and Alpine the candy uses native distro packages; on Fedora,
Debian, and Ubuntu it installs the pinned, sha256-verified `nerdctl-full` tarball
(which supplies nerdctl + containerd + `containerd-fuse-overlayfs-grpc` + CNI +
buildkit + rootlesskit in one archive). It writes `/etc/nerdctl/nerdctl.toml`
(the `charly` namespace + rootless buildkit host) and the `charly` CNI network
conflist at `/etc/cni/net.d/charly.conflist`.

## What it provides

| Property | Value |
|---|---|
| Layer / candy | `layer-nerdctl` |
| Pinned version | `NERDCTL_FULL_VERSION` `2.1.2` |
| Config | `/etc/nerdctl/nerdctl.toml`, `/etc/cni/net.d/charly.conflist` |
| Nested-pod posture | userns-scoped `cap_add: ALL`, `/dev/fuse`, `/dev/net/tun`, `unmask=/proc/*` |
| Service / port | none |

The engine word is served by `opencharly/plugin-nerdctl` (out-of-process) or
compiled in; every engine op delegates to `container.InvokeEngineOp("nerdctl", …)`.

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

- `charly.yml` — the project root: `discover:`, `defaults:`, and the
  `check-nerdctl-nested:` disposable `pod:` R10 bed.
- `candy/layer-nerdctl/charly.yml` — the `layer-nerdctl:` candy entity, its
  `security:` posture, and the embedded `layer-nerdctl-skill:` skill entity.
- `box/check-nerdctl-app/charly.yml` — the disposable bed image composing the candy.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-distros:layer-nerdctl` — the nerdctl engine, the
  `layer-nerdctl` candy, and `engine: nerdctl` deploys.
- `/charly-internals:plugin-nerdctl` — the out-of-tree engine plugin.
- `/charly-image:layer` — candy authoring reference.
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
