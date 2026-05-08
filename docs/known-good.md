# Known-Good Test State

This is the baseline to preserve before more cleanup or reset experiments.

## Latest Successful Checkpoint

```text
2026-05-08T12:19:38Z
Windows 11 vmadm install completed
```

Source state:

```text
smartos-live=vmadm-bhyve-passthru-20260505@95a08e146873894e029f620fe913e60b0f8fa7a9
illumos=vmadm-bhyve-passthru-20260505@59a39216f4a9edc31de71f30e6495ae30686463f
launcher_repo=670ad873f103
launcher=vmadm
```

Result:

```text
PASS/RUNNING
```

The user reported Windows 11 setup completed. `vmadm` showed the VM running
with `autoboot=false`. The Windows 10 VM was stopped.

## Working VM Summary

- `win11vm`
- UUID `67401f9c-1b72-4630-94eb-e7e677ce813b`
- 8 GiB RAM
- 4 vCPUs
- q35 host bridge
- flat `ppt0` through `ppt3`
- ROM attached to `ppt0`
- TPM 2.0 through `swtpm`
- `virtio1=true`
- Windows and virtio ISOs attached but non-boot
- disk boot enabled

## Preserve These Conditions

- Keep the VM on flat topology while validating changes.
- Keep `autoboot=false` unless explicitly testing boot policy.
- Keep a copy of the working UEFI VARS file before firmware or setup changes.
- Reboot the host before tests that need a clean GPU state until reset behavior
  is proven reliable.
- Record every meaningful A/B test in `/build/RUNNING_TESTS.md`.

