# Branch Matrix

This is the branch set used for the current working vmadm GPU passthrough
baseline.

| Component | Remote | Branch | Commit | Notes |
| --- | --- | --- | --- | --- |
| smartos-live | `git@github.com:dustyhair/smartos-live.git` | `vmadm-bhyve-passthru-20260505` | `95a08e146873894e029f620fe913e60b0f8fa7a9` | vmadm passthrough, TPM, virtio1, quoting fixes |
| illumos-joyent | `git@github.com:dustyhair/illumos-joyent.git` | `vmadm-bhyve-passthru-20260505` | `59a39216f4a9edc31de71f30e6495ae30686463f` | bhyve, ppt, xHCI, ACPI, immu and poll fixes |
| swtpm | `git@github.com:dustyhair/swtpm.git` | `smartos-build-support-20260507` | `08ee6b13a7aaab55c1fc4c68bd1b15d7e1b693f5` | SmartOS build support for TPM runtime |
| libtpms | `git@github.com:dustyhair/libtpms.git` | `smartos-build-support-20260507` | `9ddc2013b383f3c2e43fc910f4134c2ab069b8f4` | SmartOS build support for libtpms |
| launcher tracking | local `/build/launcher-tracking` | current local branch | `670ad873f103` | Historical standalone-launcher baseline |

## smartos-live Work

Recent relevant commits:

- `95a08e14 vmadm: quote device matches in zonecfg updates`
- `0153d453 vmadm: expose bhyve TPM option`
- `c2fe1a9a vmadm: expose bhyve virtio1 option`
- `941c3b0f bhyve vmadm: keep proc_fork privilege`
- `3508e968 vmadm: expose passthrough MSI control`
- `43017b3c vmadm: allow pci passthru device removal`
- `03675756 vmadm: allow bhyve lofs filesystems`
- `80d39089 vmadm: separate bhyve ppt device path`

Current local state when this document was written:

- `/build/smartos-live` is ahead of origin by 2 commits.
- Untracked entries exist and should not be accidentally committed:
  `.vscode/`, `ill`.

## illumos Work

Recent relevant commits:

- `59a39216f4 xhci: serialize timeout callback teardown`
- `d943bc6f1c bhyve: tolerate TPM CRB cancel writes`
- `9cf17a83c7 bhyve brand: pass TPM LPC config`
- `92cbccd1ee bhyve: log ACPI power button shutdown path`
- `3290d948b1 immu: quiet TU102-specific qinv diagnostics`
- `2a07f15d0b poll: tolerate stale pollcache bitmap entries`
- `d96c9cc35a bhyve brand: allow ACPI compiler fork`
- `027c9ac6fd bhyve: use explicit null device for iasl output`
- `d088a7b797 bhyve: run iasl without requiring a shell`
- `b5084fe8e0 bhyve brand: enable ACPI for passthru guests`

Current local state when this document was written:

- `/build/smartos-live/projects/illumos` is ahead of origin by 4 commits.
- Untracked generated files exist under `usr/src/cmd/allocate/` and should not
  be accidentally committed.

## Push State Warning

The documentation records the local working commits. If another machine needs
to reproduce this exactly from GitHub, push the smartos-live and illumos
branches first and verify the remote commit IDs match this file.

