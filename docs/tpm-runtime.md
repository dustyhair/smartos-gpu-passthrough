# TPM Runtime

Windows 11 requires TPM 2.0. The working SmartOS path uses `swtpm` with
`libtpms`.

## Source Branches

| Project | Remote | Branch | Commit |
| --- | --- | --- | --- |
| swtpm | `git@github.com:dustyhair/swtpm.git` | `smartos-build-support-20260507` | `08ee6b13a7aaab55c1fc4c68bd1b15d7e1b693f5` |
| libtpms | `git@github.com:dustyhair/libtpms.git` | `smartos-build-support-20260507` | `9ddc2013b383f3c2e43fc910f4134c2ab069b8f4` |

## Why Forks Were Needed

The upstream projects needed SmartOS portability fixes to build cleanly in this
environment. The local branches add:

- SmartOS endian handling
- missing string/header compatibility fixes
- ioctl/header portability changes
- `MIN()` fallback where needed
- avoidance of Linux/GNU-specific assumptions that do not hold on SmartOS
- build notes under `docs/smartos-build.md`

## Runtime Shape

The vmadm field is:

```text
bhyve_tpm=swtpm,/zones/build/tpm/<vm-uuid>/swtpm.sock,version=2.0
```

For the current Windows 11 VM:

```text
bhyve_tpm=swtpm,/zones/build/tpm/67401f9c-1b72-4630-94eb-e7e677ce813b/swtpm.sock,version=2.0
```

The current runtime uses staged libraries and may require `LD_LIBRARY_PATH`
matching the staged `libtpms`/`swtpm` install location.

## bhyve/SmartOS Support

SmartOS-side support is split across:

- smartos-live vmadm plumbing for `bhyve_tpm`
- illumos bhyve brand support to pass TPM LPC config
- bhyve support for TPM CRB behavior, including tolerance for cancel writes

Relevant illumos commits:

- `9cf17a83c7 bhyve brand: pass TPM LPC config`
- `d943bc6f1c bhyve: tolerate TPM CRB cancel writes`

