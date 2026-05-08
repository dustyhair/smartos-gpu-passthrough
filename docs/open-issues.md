# Open Issues

## Reset Reliability

GPU reset behavior is still an area for experimentation.

- Function-level reset has worked for the display function in some tests.
- Other functions in the GPU package may not FLR cleanly.
- Bus/device reset work exists in the branch history, but it should be tested
  carefully before relying on it for repeated VM switching.

## Interrupt Remapping

Interrupt-remapping behavior was difficult to stabilize. Avoid large unrelated
changes in the IOMMU, APIC, rootnex, or ppt interrupt paths while testing reset
behavior.

Useful areas to inspect:

```text
usr/src/uts/i86pc/io/immu_intrmap.c
usr/src/uts/i86pc/io/immu_qinv.c
usr/src/uts/i86pc/io/rootnex.c
usr/src/uts/intel/io/vmm/intel/vtd.c
usr/src/uts/intel/io/vmm/io/ppt.c
```

## xHCI Passthrough

The GPU package xHCI controller is important for keyboard and mouse passthrough
in the Windows test setup. The current branch includes teardown serialization
for xHCI endpoint timeout handling, but this path should be watched when
switching guests repeatedly.

## Windows Install Media

Windows install media may stop at a "press any key to boot from DVD" prompt.
After the first install stage, make the disk bootable and the ISO non-bootable.

## TPM Packaging

The TPM path uses SmartOS-built `swtpm` and `libtpms`. The source branches build,
but packaging and runtime library path handling should be cleaned up before
treating this as a polished SmartOS feature.

