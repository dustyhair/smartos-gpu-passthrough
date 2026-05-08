# Setup Guide

This guide assumes a SmartOS build machine that can build `smartos-live`.

## Clone smartos-live

```bash
git clone git@github.com:dustyhair/smartos-live.git
cd smartos-live
git checkout vmadm-bhyve-passthru-20260505
```

## Clone illumos-joyent

`smartos-live` expects the illumos tree under `projects/illumos`.

```bash
mkdir -p projects
git clone git@github.com:dustyhair/illumos-joyent.git projects/illumos
cd projects/illumos
git checkout vmadm-bhyve-passthru-20260505
```

Verify the expected commits:

```bash
git -C smartos-live rev-parse HEAD
git -C smartos-live/projects/illumos rev-parse HEAD
```

Expected:

```text
smartos-live: 95a08e146873894e029f620fe913e60b0f8fa7a9
illumos:      59a39216f4a9edc31de71f30e6495ae30686463f
```

## Build Environment File

The illumos build uses an environment file named `illumos.sh`. In this setup it
lives at:

```text
smartos-live/projects/illumos/illumos.sh
```

That file is usually local to the build machine and is not committed to the
illumos tree. It should point at the local SmartOS proto areas and compiler
toolchain.

Important settings from the tested environment:

```bash
NIGHTLY_OPTIONS="-CiLmMNnt"
GATE="joyent_TESTING"
CODEMGR_WS="/path/to/smartos-live/projects/illumos"
ROOT="/path/to/smartos-live/proto"
ADJUNCT_PROTO="/path/to/smartos-live/proto.strap"
NATIVE_ADJUNCT="/opt/local"
GNUC_ROOT="/path/to/smartos-live/proto.strap/usr/gcc/10"
PRIMARY_CC="gcc10,$GNUC_ROOT/bin/gcc,gnu"
PRIMARY_CCC="gcc10,$GNUC_ROOT/bin/g++,gnu"
JAVA_ROOT="/opt/local/java/openjdk11"
PYTHON3="/opt/local/bin/python3.12"
PERL="/opt/local/bin/perl"
```

If your SmartOS build machine already has a working `illumos.sh`, keep it and
adjust only the workspace paths if needed.

## TPM Sources

Windows 11 requires TPM 2.0. Clone these if you want to build the same TPM
runtime used by the tested VM:

```bash
git clone git@github.com:dustyhair/libtpms.git
git -C libtpms checkout smartos-build-support-20260507

git clone git@github.com:dustyhair/swtpm.git
git -C swtpm checkout smartos-build-support-20260507
```

See [TPM runtime](tpm-runtime.md) for details.

