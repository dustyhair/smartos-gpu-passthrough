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

Use the host's PCI tooling to identify the GPU functions. Useful commands vary
by image, but these are the usual places to start:

```bash
prtconf -dD
prtconf -v
pcitool -lv
```

Look for the NVIDIA functions and record the vendor ID and device ID. NVIDIA's
vendor ID is usually `10de`.

## Installing the Binding

There are two ways to provide `ppt_matches`:

- Build it into the platform image as `/etc/ppt_matches`.
- Place an override in the boot platform so it appears as
  `/system/boot/etc/ppt_matches` at runtime.

`libppt` intentionally checks the boot platform override first:

```text
/system/boot/etc/ppt_matches
/etc/ppt_matches
```

That means a file in the boot platform can override the file delivered inside
the normal platform image.

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

## Boot Platform Override

For fast testing, you can also place `ppt_matches` directly into the boot
platform `etc` directory used by the machine before reboot. On the running
SmartOS host, that file is seen as:

```text
/system/boot/etc/ppt_matches
```

On a SmartOS boot server or staging area this is commonly the `platform/etc`
directory. Example layout:

```text
platform/etc/ppt_matches
platform-<stamp>/etc/ppt_matches
```

If your environment uses a path such as `/zones/boot`, copy the same file into
the active platform directories before reboot:

```bash
mkdir -p /zones/boot/platform/etc
cp ppt_matches /zones/boot/platform/etc/ppt_matches

mkdir -p /zones/boot/platform-TESTING/etc
cp ppt_matches /zones/boot/platform-TESTING/etc/ppt_matches
```

Adjust `platform-TESTING` to whatever platform stamp your test image uses.

This boot-platform copy is useful because it avoids rebuilding the platform just
to change which PCI IDs bind to `ppt`.

## ppt_aliases

Some older or local workflows also stage an empty or generated
`ppt_aliases` file alongside `ppt_matches`:

```text
platform/etc/ppt_aliases
platform-<stamp>/etc/ppt_aliases
```

`ppt_matches` is the important file for selecting devices by vendor/device ID.
Only add `ppt_aliases` if your branch or local tooling expects it. If you do
add it to a built image, it also needs a manifest entry, for example:

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
cat /system/boot/etc/ppt_matches 2>/dev/null || cat /etc/ppt_matches
```

The guest configuration should use the resulting `/dev/pptN` paths. Do not
assume that `ppt0` is always the display function; verify the mapping on each
host.
