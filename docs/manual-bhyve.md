# Manual bhyve Launch

`vmadm` is the preferred path for repeatable testing, but a direct bhyve launch
is useful for A/B testing and reducing the stack.

This example mirrors the known-good flat topology. Adjust paths and device names
for your system.

## Device Assumptions

```text
/dev/ppt0  GPU display function
/dev/ppt1  GPU audio function
/dev/ppt2  GPU xHCI function
/dev/ppt3  GPU UCSI / auxiliary function
```

Verify this with `pptadm list` before launching.

## Command Shape

```bash
/usr/sbin/bhyvectl --create --vm=win11-gpu

/usr/sbin/bhyve \
  -c 4 \
  -m 8G \
  -w -H -S -P -a -A -Y \
  -l bootrom,/path/to/BHYVE_UEFI_CODE.fd,/path/to/BHYVE_VARS_WIN11.fd \
  -s 0:0,hostbridge,model=q35 \
  -s 0:8:0,passthru,/dev/ppt0,rom=/path/to/gpu.rom,rom-exec=off \
  -s 0:8:1,passthru,/dev/ppt1 \
  -s 0:8:2,passthru,/dev/ppt2 \
  -s 0:8:3,passthru,/dev/ppt3 \
  -s 4:0,ahci-hd,/dev/zvol/rdsk/zones/win11-disk0 \
  -s 3:0,ahci-cd,/path/to/virtio-win.iso \
  -s 3:1,ahci-cd,/path/to/Win11.iso \
  -s 5:0,xhci,tablet \
  -s 6:0,virtio-net-viona,vnic=bhyve0 \
  -s 31,lpc \
  -o lpc.fwcfg=qemu \
  -f name=opt/gpu-diag,file=/path/to/fwcfg-gpu-diag.txt \
  win11-gpu
```

## Important Options

| Option | Purpose |
| --- | --- |
| `-m 8G` | Give Windows enough RAM. Avoid 1 GiB passthrough tests. |
| `-s 0:0,hostbridge,model=q35` | Use q35 host bridge. |
| `-a` | Disable x2APIC for the tested Windows path. |
| `-A` | Enable ACPI. |
| `-Y` | Disable MPTable. |
| `-l bootrom,code,vars` | Use UEFI firmware with a writable vars file. |
| `rom=/path/to/gpu.rom` | Supply the NVIDIA option ROM. |
| `rom-exec=off` | Keep ROM mapped without executing it after firmware use. |
| `-o lpc.fwcfg=qemu` | Enable qemu-style fwcfg used by the firmware path. |
| `-f name=opt/gpu-diag,file=...` | Optional fwcfg file used by the tested firmware path. |

## TPM

Manual TPM wiring depends on the bhyve branch and local `swtpm` launch method.
For Windows 11, the `vmadm` path is easier because `bhyve_tpm` records the TPM
socket in the VM config and the brand passes it through consistently.

## Cleanup

Destroy the VM after a manual test:

```bash
/usr/sbin/bhyvectl --destroy --vm=win11-gpu
```
