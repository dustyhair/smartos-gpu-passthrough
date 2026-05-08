# SmartOS GPU Passthrough

This repository documents the branch set and setup steps used to experiment
with NVIDIA GPU passthrough on SmartOS bhyve.

The current working direction is **bhyve GPU passthrough through `vmadm`**. The
goal is to make it easy to clone the right trees, check out the right branches,
build SmartOS, and try the different passthrough branches without reconstructing
the history from chat logs.

## What Works

- Windows 11 can install and boot with GPU passthrough through `vmadm`.
- The tested GPU is an NVIDIA TU102 device, specifically an RTX 2080 Ti class
  card.
- The working layout uses flat PCI topology.
- The tested VM passes through all GPU package functions:
  - display function
  - HDMI/DP audio
  - USB xHCI controller
  - UCSI / auxiliary function
- Windows 11 uses TPM 2.0 through `swtpm`.
- The GPU ROM is supplied to the display function, with ROM execution disabled
  after firmware use.

## Start Here

1. Read the [branch matrix](docs/branches.md).
2. Follow the [setup guide](docs/setup.md).
3. Build with the [build notes](docs/build.md).
4. Create a test VM using [vmadm Windows notes](docs/vmadm-windows.md).
5. Use [known-good settings](docs/known-good.md) as the first target.
6. Review [open issues](docs/open-issues.md) before changing reset, topology, or
   interrupt-remapping behavior.

## Repository Layout

- `docs/branches.md`: branch and commit map.
- `docs/setup.md`: how to clone the trees and select branches.
- `docs/build.md`: SmartOS/illumos build notes.
- `docs/vmadm-windows.md`: example VM configuration for Windows passthrough.
- `docs/tpm-runtime.md`: `swtpm` and `libtpms` notes.
- `docs/known-good.md`: baseline settings known to boot.
- `docs/open-issues.md`: known limitations and areas still under test.

