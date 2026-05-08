# vmadm Windows Passthrough Example

This is an example shape for a Windows 11 passthrough VM. UUIDs, disk names,
device names, and ROM paths should be adjusted for the local machine.

## VM Identity

- Alias: `win11vm`
- UUID: local VM UUID
- RAM: `8192` MiB
- vCPUs: `4`
- `autoboot`: `false`
- Host bridge: `q35`
- Topology: flat

## Firmware

```text
bootrom=/path/to/BHYVE_UEFI_CODE.fd,/path/to/BHYVE_VARS_WIN11.fd
```

Extra bhyve options:

```text
-w -P -a -Y -o lpc.fwcfg=qemu -f name=opt/gpu-diag,file=/path/to/fwcfg-gpu-diag.txt
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

- Windows 11 installer ISO, `boot=false`, `ahci-cd`, PCI slot `0:2:0`
- virtio driver ISO, `boot=false`, `ahci-cd`, PCI slot `0:3:0`

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
/path/to/gpu.rom
```

## TPM

TPM is required for Windows 11 and is provided by `swtpm`:

```text
bhyve_tpm=swtpm,/path/to/tpm/<vm-uuid>/swtpm.sock,version=2.0
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

- Keep `autoboot=false` while experimenting.
- After first-stage Windows install, set the disk as bootable and set the
  installer ISO as non-bootable to avoid the DVD "press any key" path.
