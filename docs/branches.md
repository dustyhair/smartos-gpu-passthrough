# Branch Matrix

These branches are the current known-good set for trying GPU passthrough with
`vmadm`.

| Component | Repository | Branch | Commit |
| --- | --- | --- | --- |
| smartos-live | `git@github.com:dustyhair/smartos-live.git` | `vmadm-bhyve-passthru-20260505` | `95a08e146873894e029f620fe913e60b0f8fa7a9` |
| illumos-joyent | `git@github.com:dustyhair/illumos-joyent.git` | `vmadm-bhyve-passthru-20260505` | `59a39216f4a9edc31de71f30e6495ae30686463f` |
| swtpm | `git@github.com:dustyhair/swtpm.git` | `smartos-build-support-20260507` | `08ee6b13a7aaab55c1fc4c68bd1b15d7e1b693f5` |
| libtpms | `git@github.com:dustyhair/libtpms.git` | `smartos-build-support-20260507` | `9ddc2013b383f3c2e43fc910f4134c2ab069b8f4` |

## Branch Purpose

`smartos-live: vmadm-bhyve-passthru-20260505`

Adds the `vmadm` plumbing needed to describe passthrough VMs:

- GPU ROM support for bhyve passthrough devices.
- Per-device passthrough path handling.
- passthrough device removal.
- passthrough MSI control.
- `bhyve_virtio1`.
- `bhyve_tpm`.
- quoting fixes for zonecfg device matches.

`illumos-joyent: vmadm-bhyve-passthru-20260505`

Carries the kernel, bhyve, and bhyve-brand work:

- bhyve passthrough ROM handling.
- ppt assignment, teardown, reset, and MSI support.
- interrupt-remapping fixes for passthrough.
- Intel VT-d / IOMMU behavior needed by the passthrough path.
- ACPI generation fixes for passthrough guests.
- TPM LPC/CRB support for Windows 11.
- xHCI timeout teardown fixes.
- reduced TU102-specific diagnostic logging.

`swtpm` and `libtpms`

Carry SmartOS portability fixes needed to build and run a TPM 2.0 provider for
Windows 11.

## Older Experiment Branches

These branches may be useful when bisecting behavior or comparing earlier
approaches:

| Branch | Repository | Use |
| --- | --- | --- |
| `gpu-passthrough-minimal` | illumos-joyent | earlier minimal passthrough baseline |
| `gpu-passthrough-immu` | illumos-joyent | interrupt-remapping / IOMMU work |
| `gpu-passthrough-late-runtime` | illumos-joyent | late runtime debugging |
| `ppt-vfio-lifecycle` | illumos-joyent | ppt lifecycle experiments |
| `ppt-lifecycle-refactor` | illumos-joyent | lifecycle refactor work |

For normal testing, start with `vmadm-bhyve-passthru-20260505`.

