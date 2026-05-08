# Known-Good Baseline

Use this as the first target when bringing up a new test system.

## Source State

```text
smartos-live branch: vmadm-bhyve-passthru-20260505
smartos-live commit: 95a08e146873894e029f620fe913e60b0f8fa7a9

illumos branch: vmadm-bhyve-passthru-20260505
illumos commit: 59a39216f4a9edc31de71f30e6495ae30686463f
```

## Guest Shape

- Windows 11.
- 8 GiB RAM.
- 4 vCPUs.
- q35 host bridge.
- flat passthrough topology.
- virtio1 enabled.
- TPM 2.0 through `swtpm`.
- GPU ROM supplied to the display function.
- installer ISO and virtio driver ISO attached as non-boot devices after
  installation.

## Passthrough Device Set

The tested GPU package was passed as four functions:

| Function | Example device | Notes |
| --- | --- | --- |
| display | `ppt0` | ROM attached, ROM execution disabled after firmware use |
| audio | `ppt1` | same GPU package |
| xHCI | `ppt2` | used for USB keyboard/mouse path |
| UCSI / auxiliary | `ppt3` | same GPU package |

## Windows 11 Setup

If Windows setup cannot find a network driver, bypass network during OOBE:

```text
Shift+F10
OOBE\BYPASSNRO
```

Install virtio and NVIDIA drivers after Windows reaches the desktop.

