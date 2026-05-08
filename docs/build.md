# Build Notes

This branch set is intended to build through the normal SmartOS build flow.

## Build smartos-live

From the `smartos-live` root:

```bash
gmake live
```

This should build illumos through the SmartOS build machinery and produce the
usual SmartOS live image artifacts under:

```text
smartos-live/output/
```

## Full Rebuild

If you want to force a full illumos rebuild, remove the illumos stamp before
running `gmake live`:

```bash
rm -f 0-illumos-stamp
gmake live
```

Some local trees may use a stamp named `0-illumos`; check the repository root if
you are not sure.

## Incremental Development

During development it is useful to build only changed illumos areas inside
`bldenv`, but a final validation should use a normal `gmake live` from
`smartos-live`.

Common areas touched by this work:

```text
usr/src/cmd/bhyve
usr/src/lib/brand/bhyve
usr/src/uts/intel/io/vmm
usr/src/uts/i86pc/io/immu*
usr/src/uts/i86pc/io/rootnex*
usr/src/uts/common/io/usb/hcd/xhci
```

## Boot Artifacts

The passthrough work should not require custom boot loader changes. Prefer
testing platform/live image changes first. Only test boot archive or boot loader
changes if you are deliberately working on that part of the stack.

## Build Stamp

The current `smartos-live` branch forces the build stamp to `TESTING`. This is
convenient for repeated local testing but is not a general release practice.

