# ppt Binding

SmartOS passthrough uses the `ppt` driver to reserve host PCI devices for bhyve.
A device must bind to `ppt` on the host before it can be passed to a guest as
`/dev/pptN`.

## ppt_matches

`ppt_matches` lists PCI vendor/device IDs that should attach to `ppt`.

The format is one device ID per line:

```text
pci<vendor>,<device>
```

Example shape:

```text
pci10de,1e07
pci10de,10f7
pci10de,1ad6
pci10de,1ad7
```

Those example IDs represent the four functions commonly seen on an NVIDIA GPU
package: display, audio, USB xHCI, and UCSI. Use the IDs from your own card, not
the example blindly.

## Finding Device IDs

Identify the GPU while it is still attached to its normal host drivers, then
bind those functions to `ppt`, reboot, and use `pptadm` to map `/dev/pptN`
devices back to the physical functions.

The source of truth is the host device tree:

| Question | Command | What it proves |
| --- | --- | --- |
| What PCI vendor/device IDs exist? | `prtconf -d` | Shows PCI IDs such as `pci10de,1e07`. |
| What driver owns the device now? | `prtconf -D` | Shows whether the function is still on a normal host driver or already on `ppt`. |
| What exact device-tree path is this function? | `prtconf -v` | Shows the physical `/devices` tree and properties for the node. |
| Which `/dev/pptN` did it become after binding? | `pptadm list -a -o all` or `pptadm list -j -a` | Maps `/dev/pptN` back to physical path and IDs. |

The goal is to prove this chain:

```text
physical GPU slot/function -> PCI vendor/device ID -> ppt_matches entry -> same physical path under pptadm -> /dev/pptN -> guest slot
```

## Before Binding: Find the GPU Functions

Use the host's PCI tooling before the device is reserved by `ppt`:

```bash
prtconf -dD
prtconf -v
```

Start with `prtconf -dD`. The `-d` option prints PCI vendor/device IDs, and the
`-D` option prints the attached driver. Look for a group of NVIDIA functions on
the same PCI package. NVIDIA's vendor ID is usually `10de`. A TU102 / RTX 2080
Ti style card commonly has four functions:

| Function | Class | What to look for |
| --- | --- | --- |
| display | VGA / 3D / display controller | NVIDIA display function |
| audio | HD audio controller | NVIDIA HDMI/DP audio |
| xHCI | USB controller | NVIDIA USB 3.x / xHCI |
| UCSI / aux | serial bus / USB-C related | NVIDIA UCSI / auxiliary function |

The functions usually share the same PCI slot and differ only by function
number, for example:

```text
07:00.0  display
07:00.1  audio
07:00.2  xHCI
07:00.3  UCSI / auxiliary
```

On illumos/SmartOS, the stable thing to record is usually the physical device
tree path, not just the short BDF. `prtconf -v` shows the parent bridges and
the child function nodes. The sibling functions should appear below the same
PCI bridge path and should differ only by the final function node.

Example shape:

```text
/pci@0,0/.../pci10de,1e07@0
/pci@0,0/.../pci10de,10f7@0,1
/pci@0,0/.../pci10de,1ad6@0,2
/pci@0,0/.../pci10de,1ad7@0,3
```

The exact path will differ by motherboard and slot. The important part is that
all functions are siblings under the same bridge path and have NVIDIA IDs. That
is how you know they are the same physical card package.

Record the vendor/device IDs for each function. Convert each pair into a
`ppt_matches` line:

```text
vendor=10de device=1e07 -> pci10de,1e07
vendor=10de device=10f7 -> pci10de,10f7
vendor=10de device=1ad6 -> pci10de,1ad6
vendor=10de device=1ad7 -> pci10de,1ad7
```

The exact IDs vary by card, board vendor, and generation. Do not copy these
unless they match your hardware.

If the host has more than one NVIDIA device, prefer matching by the exact
physical path if you only want one card reserved. `pptadm(8)` supports matches
by PCI ID or physical path; PCI ID matches reserve every matching device.

## How to Know It Is the Correct Device

Use multiple independent checks before binding a device to `ppt`:

1. Confirm the vendor/device IDs with `prtconf -dD`.
2. Confirm the functions are siblings in the same physical path with
   `prtconf -v`.
3. Confirm the function classes match the expected GPU package layout:
   display, audio, xHCI, and UCSI/auxiliary.
4. Confirm the currently attached driver is plausible before binding. The
   display function should not already be an unrelated storage, network, or
   chipset driver.
5. If there are multiple matching GPUs, use physical-path matching or remove
   ambiguity before binding. A broad `pci10de,...` match binds every device
   with that ID.
6. After reboot, confirm `pptadm` shows the same physical paths and IDs that
   you recorded before binding.

For a single-card passthrough setup, the strongest check is the before/after
path match:

```text
before: prtconf -v shows /pci@0,0/.../pci10de,1e07@0
match:  ppt_matches contains pci10de,1e07 or the exact physical path
after:  pptadm list -a -o all shows /dev/ppt0 with /pci@0,0/.../pci10de,1e07@0
```

If the post-binding `pptadm` path does not match the pre-binding `prtconf -v`
path, stop and fix the binding before starting a guest. Do not guess from
`ppt0`, `ppt1`, or creation order.

## After Binding: Map ppt Devices

After `ppt_matches` is installed and the host has rebooted, use:

```bash
pptadm list -a -o all
pptadm list -j -a
```

Useful `pptadm` fields:

| Field | Use |
| --- | --- |
| `dev` | `/dev/pptN` path to use in bhyve/vmadm. |
| `path` | Physical `/devices` path. Use this to identify slot/function. |
| `vendor` / `vendor-id` | PCI vendor ID. |
| `device` / `device-id` | PCI device ID. |
| `subvendor` / `subsystem-vendor-id` | Board vendor ID. |
| `subdevice` / `subsystem-id` | Board-specific subsystem ID. |
| `label` | Human-readable PCI database label. |

Build a simple map before creating the VM:

```text
/dev/ppt0 -> display -> guest 0:8:0
/dev/ppt1 -> audio   -> guest 0:8:1
/dev/ppt2 -> xHCI    -> guest 0:8:2
/dev/ppt3 -> UCSI    -> guest 0:8:3
```

Do not assume the `pptN` order. Confirm it with `pptadm list -j -a` after every
hardware change or binding change.

## Cross-Checking Guest Slots

The known-good flat topology keeps the GPU package as one multifunction guest
device:

```text
0:8:0  display
0:8:1  audio
0:8:2  xHCI
0:8:3  UCSI / auxiliary
```

Use that map in `vmadm`:

```json
{
  "path": "/dev/ppt0",
  "pptdev": "ppt0",
  "pci_slot": "0:8:0",
  "rom": "/path/to/gpu.rom",
  "rom_exec": false
}
```

The other functions use the same slot with function numbers `1`, `2`, and `3`.

## Installing the Binding

There are two ways to provide `ppt_matches`:

- Build it into the platform image as `/etc/ppt_matches`.
- Load it as a boot file module from `/boot/etc/ppt_matches`.

`libppt` intentionally checks the boot-module provided file first:

```text
/system/boot/etc/ppt_matches
/etc/ppt_matches
```

That means a loader-provided file can override the file delivered inside the
normal platform image.

## Source Build Path

For a source build, the default source file is:

```text
smartos-live/projects/illumos/usr/src/lib/libppt/ppt_matches
```

The install rule is in:

```text
smartos-live/projects/illumos/usr/src/lib/libppt/Makefile
```

The important pieces are:

```make
ETCFILES=	ppt_matches
ROOTETC=	$(ROOT)/etc
```

During an illumos install, this copies `ppt_matches` into:

```text
smartos-live/proto/etc/ppt_matches
```

For SmartOS live image packaging, make sure the file is present in the platform
manifest:

```text
f etc/ppt_matches 0444 root root
```

In this tree that line is present in:

```text
smartos-live/manifest.d/illumos.manifest
smartos-live/manifest.gen
```

After editing the source `ppt_matches`, rebuild illumos and run `gmake live` so
the new platform image includes it.

## Loader Boot Module Override

For the tested setup, `ppt_matches` and `ppt_aliases` are loaded by the boot
loader as file modules. This is the practical way to change passthrough binding
without rebuilding the whole platform image.

Place the files in the boot tree:

```text
/boot/etc/ppt_matches
/boot/etc/ppt_aliases
```

Then add these lines to `loader.conf`:

```text
ppt_aliases_load="YES"
ppt_aliases_type="file"
ppt_aliases_name="/boot/etc/ppt_aliases"
ppt_aliases_flags="name=/etc/ppt_aliases"
ppt_matches_load="YES"
ppt_matches_type="file"
ppt_matches_name="/boot/etc/ppt_matches"
ppt_matches_flags="name=/etc/ppt_matches"
```

The `_name` value is where the loader reads the file from. The `_flags`
`name=/etc/...` value is the boot module name illumos uses. At runtime, that
boot module is visible through `/system/boot`, so `libppt` sees:

```text
/system/boot/etc/ppt_matches
/system/boot/etc/ppt_aliases
```

If your SmartOS boot tree is staged somewhere else, put the same files under
that boot tree's `boot/etc` directory. For example, in a staging area:

```bash
mkdir -p /path/to/boot-tree/boot/etc
cp ppt_matches /path/to/boot-tree/boot/etc/ppt_matches
cp ppt_aliases /path/to/boot-tree/boot/etc/ppt_aliases
```

Also make sure the `loader.conf` used by that boot tree contains the file-module
entries above.

## ppt_aliases

The tested loader setup loads `ppt_aliases` alongside `ppt_matches`:

```text
/boot/etc/ppt_aliases
```

`ppt_matches` is the file that selects devices by vendor/device ID. `ppt_aliases`
is used for driver alias style bindings when present. If you add `ppt_aliases`
to a built platform image rather than loading it from the boot tree, it also
needs a manifest entry, for example:

```text
f etc/ppt_aliases 0444 root root
```

## Verifying

After boot, verify that the devices are owned by `ppt`:

```bash
pptadm list -a
pptadm list -j
```

Also confirm which file is visible:

```bash
ls -l /system/boot/etc/ppt_matches /etc/ppt_matches
ls -l /system/boot/etc/ppt_aliases /etc/ppt_aliases
cat /system/boot/etc/ppt_matches 2>/dev/null || cat /etc/ppt_matches
```

The guest configuration should use the resulting `/dev/pptN` paths. Do not
assume that `ppt0` is always the display function; verify the mapping on each
host.
