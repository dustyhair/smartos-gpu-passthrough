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
