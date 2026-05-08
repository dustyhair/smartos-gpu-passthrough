# vmadm Support

The `vmadm-bhyve-passthru-20260505` branch adds enough bhyve passthrough
plumbing to describe a GPU VM through normal SmartOS VM configuration.

## Added or Extended Properties

Top-level bhyve properties:

| Property | Purpose |
| --- | --- |
| `bhyve_tpm` | Pass a TPM backend to the bhyve brand. Used for Windows 11. |
| `virtio1` | Select legacy/transitional virtio behavior for guests that need it. |
| `bhyve_extra_opts` | Pass additional bhyve command-line options. |
| `bhyve_hostbridge` | Select host bridge model, such as `q35`. |

PCI passthrough properties:

| Property | Purpose |
| --- | --- |
| `pci_devices` | Full passthrough device list on create. |
| `add_pci_devices` | Add passthrough devices during update. |
| `update_pci_devices` | Update passthrough device metadata. |
| `remove_pci_devices` | Remove passthrough devices during update. |
| `pci_devices.*.path` | Host `/dev/pptN` path. |
| `pci_devices.*.pptdev` | Stable logical name such as `ppt0`. |
| `pci_devices.*.pci_slot` | Guest PCI slot/function, for example `0:8:0`. |
| `pci_devices.*.rom` | GPU option ROM file for the display function. |
| `pci_devices.*.rom_exec` | Whether bhyve should execute the ROM. |
| `pci_devices.*.msi` | Optional passthrough MSI control. |

## Example Create Payload

This is a minimal shape, not a complete production manifest. Adjust paths,
UUIDs, disks, NICs, and package values for your environment.

```json
{
  "brand": "bhyve",
  "alias": "win11-gpu",
  "autoboot": false,
  "ram": 8192,
  "vcpus": 4,
  "bhyve_hostbridge": "q35",
  "virtio1": true,
  "bootrom": "/path/to/BHYVE_UEFI_CODE.fd,/path/to/BHYVE_VARS_WIN11.fd",
  "bhyve_extra_opts": "-w -P -a -Y -o lpc.fwcfg=qemu",
  "bhyve_tpm": "swtpm,/path/to/tpm/<vm-uuid>/swtpm.sock,version=2.0",
  "disks": [
    {
      "path": "/dev/zvol/rdsk/zones/win11-disk0",
      "media": "disk",
      "model": "ahci",
      "boot": true,
      "pci_slot": "0:4:0"
    },
    {
      "path": "/path/to/Win11.iso",
      "media": "cdrom",
      "model": "ahci",
      "boot": false,
      "pci_slot": "0:2:0"
    },
    {
      "path": "/path/to/virtio-win.iso",
      "media": "cdrom",
      "model": "ahci",
      "boot": false,
      "pci_slot": "0:3:0"
    }
  ],
  "nics": [
    {
      "nic_tag": "admin",
      "model": "virtio"
    }
  ],
  "pci_devices": [
    {
      "path": "/dev/ppt0",
      "pptdev": "ppt0",
      "pci_slot": "0:8:0",
      "rom": "/path/to/gpu.rom",
      "rom_exec": false
    },
    {
      "path": "/dev/ppt1",
      "pptdev": "ppt1",
      "pci_slot": "0:8:1"
    },
    {
      "path": "/dev/ppt2",
      "pptdev": "ppt2",
      "pci_slot": "0:8:2"
    },
    {
      "path": "/dev/ppt3",
      "pptdev": "ppt3",
      "pci_slot": "0:8:3"
    }
  ]
}
```

Create it with:

```bash
vmadm create < win11-gpu.json
```

## Updating Devices

Use `add_pci_devices`, `update_pci_devices`, and `remove_pci_devices` rather
than replacing the whole VM when changing passthrough attachments.

Example add shape:

```json
{
  "add_pci_devices": [
    {
      "path": "/dev/ppt2",
      "pptdev": "ppt2",
      "pci_slot": "0:8:2"
    }
  ]
}
```

Apply it with:

```bash
vmadm update <vm-uuid> < add-ppt2.json
```

Example remove shape:

```json
{
  "remove_pci_devices": ["ppt2"]
}
```

Apply it with:

```bash
vmadm update <vm-uuid> < remove-ppt2.json
```

## Flat Topology

The known-good configuration keeps all GPU functions on one guest slot:

```text
0:8:0  display
0:8:1  audio
0:8:2  xHCI
0:8:3  UCSI / auxiliary
```

This mirrors a multifunction PCI device and avoids adding synthetic root ports
while validating passthrough behavior.
