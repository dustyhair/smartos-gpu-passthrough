# SmartOS GPU Passthrough Notes

This repository records the working SmartOS GPU passthrough baseline, the
branches needed to reproduce it, and the operational details that were learned
while making NVIDIA TU102 passthrough work under `vmadm`.

The current known-good path is **SmartOS bhyve through vmadm**, not the older
standalone launcher path. The old launcher remains useful as a historical A/B
baseline, but the active goal is to keep passthrough reproducible through normal
SmartOS VM management.

## Current Status

- Windows 11 installs and boots through `vmadm`.
- The VM uses flat PCI topology with GPU display, audio, xHCI, and auxiliary
  functions attached as `ppt0` through `ppt3`.
- TPM 2.0 is provided by `swtpm`.
- The NVIDIA GPU ROM is attached to `ppt0`; `rom_exec` is disabled.
- The VM uses 8 GiB RAM and 4 vCPUs.
- `autoboot` is disabled while testing.

## Documentation Map

- [Branch matrix](docs/branches.md)
- [Build and deploy workflow](docs/build-deploy.md)
- [Known-good vmadm Windows 11 configuration](docs/vmadm-windows.md)
- [TPM runtime and swtpm/libtpms notes](docs/tpm-runtime.md)
- [Known-good test state](docs/known-good.md)
- [Open issues and risks](docs/open-issues.md)

## Non-Negotiable Test Rules

- Do not install or modify `boot-TESTING` unless there is explicit approval for
  that specific action.
- Treat a working deployed image and a working source commit as different until
  a clean rebuild and retest proves they match.
- Record meaningful builds, deploys, and runtime tests in
  `/build/RUNNING_TESTS.md`.
- Use at least 4 GiB RAM for bhyve passthrough tests. The working Windows
  baseline uses 8 GiB.
- Keep passthrough topology flat unless a test explicitly says otherwise.

