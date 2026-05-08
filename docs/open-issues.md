# Open Issues and Risks

## Branches Not Fully Published Yet

At the time this repository was initialized:

- `/build/smartos-live` was ahead of origin by 2 commits.
- `/build/smartos-live/projects/illumos` was ahead of origin by 4 commits.

Push both branches before treating the GitHub branch matrix as reproducible from
a fresh clone.

## Residual Runtime Questions

- The Windows 11 setup window logged an `ahci` watchdog and zone init restart,
  but no new host panic was observed.
- `ppt0` FLR has worked, while `ppt1` through `ppt3` have returned `EIO`.
  Bus/device reset work remains future testing.
- GPU reset reliability is not yet proven enough to skip host reboots before
  high-signal passthrough tests.
- The old launcher path is useful for A/B testing but should not be the primary
  operational path now that vmadm can boot the VM.

## Firmware and Install Friction

- Windows install media can stop at the DVD "press any key" prompt. After the
  first install stage, keep the disk bootable and installer ISO non-bootable.
- A no-prompt Windows 11 ISO has not been built in this baseline.
- Keep working UEFI VARS files backed up before making firmware or boot-order
  changes.

## TPM Runtime Cleanup

- `swtpm` currently relies on locally staged SmartOS builds.
- Confirm the runtime library path and packaging strategy before calling TPM
  support production-ready.

## Logging Cleanup

Most TU102-specific diagnostics were removed or quieted, but new debugging work
should avoid adding permanent device-specific log spam. If instrumentation is
needed, make it gated, short-lived, or clearly marked for removal.

