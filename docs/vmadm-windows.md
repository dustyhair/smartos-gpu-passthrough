# vmadm Windows Passthrough Baseline

The current known-good VM is Windows 11 managed by `vmadm`.

## VM Identity

- Alias: `win11vm`
- UUID: `67401f9c-1b72-4630-94eb-e7e677ce813b`
- RAM: `8192` MiB
- vCPUs: `4`
- `autoboot`: `false`
- Host bridge: `q35`
- Topology: flat

## Firmware

```text
/zones/build/fw/BHYVE_UEFI_CODE.fd
/zones/build/BHYVE_VARS_WIN11VM_VMADM_WORK.fd
```

Configured as:

```text
bootrom=/zones/build/fw/BHYVE_UEFI_CODE.fd,/zones/build/BHYVE_VARS_WIN11VM_VMADM_WORK.fd
```

Extra bhyve options:

```text
-w -P -a -Y -o lpc.fwcfg=qemu -f name=opt/gpu-diag,file=/zones/build/fwcfg-gpu-diag.txt
```

## Storage

Primary disk:

```text
/dev/zvol/rdsk/zones/win11-disk0
```

Properties:

- type: `ahci`
- boot: `true`
- size: `81920` MiB
- PCI slot: `0:4:0`

Attached ISOs after installation:

- `/zones/build/Win11_25H2_English_x64_v2.iso`, `boot=false`, `ahci-cd`,
  PCI slot `0:2:0`
- `/zones/build/virtio-win.iso`, `boot=false`, `ahci-cd`, PCI slot `0:3:0`

## Passthrough Devices

Flat topology device layout:

| Device | Function | Guest slot | Notes |
| --- | --- | --- | --- |
| `ppt0` | GPU display | `0:8:0` | ROM attached, `rom_exec=false` |
| `ppt1` | GPU audio | `0:8:1` | Same multifunction group |
| `ppt2` | xHCI | `0:8:2` | Required for keyboard/mouse path |
| `ppt3` | UCSI / auxiliary | `0:8:3` | Same physical GPU package |

GPU ROM:

```text
/zones/build/MSI.RTX2080Ti.1e07.raw.rom
```

## TPM

TPM is required for Windows 11 and is provided by `swtpm`:

```text
bhyve_tpm=swtpm,/zones/build/tpm/67401f9c-1b72-4630-94eb-e7e677ce813b/swtpm.sock,version=2.0
```

## Network

The VM uses a virtio NIC on the `admin` network. Windows setup may not have the
virtio network driver available during OOBE.

The successful Windows 11 setup used:

```text
Shift+F10
OOBE\BYPASSNRO
```

After reboot, continue setup without network and install drivers later.

## Operational Notes

- Use JSON on stdin for `vmadm update`; avoid passing large JSON as a shell
  argument.
- Confirm `autoboot=false` after edits.
- Do not leave keyboard injectors or boot automation loops running after use.
- After first-stage Windows install, set the disk as bootable and the installer
  ISO as non-bootable to avoid the DVD "press any key" path.

