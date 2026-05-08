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

For a source build, the default file is:

```text
smartos-live/projects/illumos/usr/src/lib/libppt/ppt_matches
```

After editing it, rebuild and install a new SmartOS image.

For a deployed SmartOS image, the file must be present in the platform's `/etc`
area before the host boots. The installed path inside the running platform is:

```text
/etc/ppt_matches
```

At runtime, `libppt` checks the boot platform copy first and then falls back to
the live `/etc` copy:

```text
/system/boot/etc/ppt_matches
/etc/ppt_matches
```

If your workflow stages platform files before reboot, make sure `ppt_matches`
is copied into the staged platform `etc` directory so it is present at boot.

## Verifying

After boot, verify that the devices are owned by `ppt`:

```bash
pptadm list -a
pptadm list -j
```

The guest configuration should use the resulting `/dev/pptN` paths. Do not
assume that `ppt0` is always the display function; verify the mapping on each
host.
