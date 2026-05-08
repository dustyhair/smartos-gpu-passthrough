# Build and Deploy Workflow

This workflow is for the builder zone at `/build`.

## Environment

Use the SmartOS builder environment, not an arbitrary Linux shell.

```bash
source /build/.profile >/dev/null 2>&1 || true
cd /build/smartos-live/projects/illumos
/build/bin/buildenv
```

Use existing helpers before adding new automation:

- `/build/bin/smartbuild`
- `/build/bin/sendit`
- `/build/bin/diffit`
- `/build/bin/gpu-state-snapshot`
- `/build/bin/gpu-host-baseline`
- `/build/bin/ppt-gpu-mode`

## Incremental Builds

`smartbuild` maps changed areas to the right illumos targets. Examples:

```bash
/build/bin/smartbuild --no-auto --module 'uts/intel ppt' --install
/build/bin/smartbuild --no-auto --module 'uts/i86pc vmm' --install
/build/bin/smartbuild --no-auto --module 'cmd bhyve' --install
/build/bin/smartbuild --no-auto --module 'uts/intel xhci' --install
```

For a live image after incremental work:

```bash
cd /build/smartos-live
gmake live
```

## Deploy

Preferred deploy path:

```bash
/build/bin/smartbuild --no-auto --install --live --push
```

or, after an already completed live image:

```bash
/build/bin/sendit
```

`sendit` copies the `*TESTING.tgz` output to `192.168.1.201`, installs the
platform image, syncs `ppt_matches` and `ppt_aliases`, and reboots the test
machine.

## Boot Tree Guardrail

Do not install or modify `boot-TESTING`, `/zones/boot/boot-TESTING`, or any
boot archive / boot tree on `192.168.1.201` unless explicitly approved for that
specific action. Before doing so, write the justification, risk, and expected
effect, then wait for permission.

The tested passthrough workflow should use platform image deploys and avoid boot
tree changes.

## Reproducibility Checklist

Before every meaningful build or deploy, record:

- smartos-live branch and commit
- illumos branch and commit
- whether tracked local edits exist
- whether untracked files exist
- launcher repo commit, if the launcher path is used
- which modules were rebuilt into `proto`

Then add a concise entry to `/build/RUNNING_TESTS.md`.

