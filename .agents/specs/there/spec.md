# `there` Specification

## Purpose

`there` is a thin project-local wrapper around Lima. It gives the current directory a dedicated Linux virtual machine,
mounts that directory inside the guest, and provides a small command surface for entering, running commands in, stopping,
destroying, and pruning Lima state.

The current implementation targets Ubuntu by default.

## Execution Model

`there` always operates on the caller's physical current directory, as returned by `pwd -P`.

For each supported image key and host path pair, `there` derives one Lima instance name:

```text
<image-key>-<basename-of-physical-path>-<first-6-hex-chars-of-md5(path)>
```

The hash input is the full physical path string. This makes instance names stable across repeated invocations from the
same directory while reducing collisions between directories that have the same basename.

The current directory is mounted in the guest as `/here` with write access. Commands executed through `there run` and
interactive shells entered through `there` or `there start` use `/here` as the guest working directory.

## Image Selection

`there` determines the image key from the executable name:

- If invoked as `there`, the image key is `ubuntu`.
- If invoked through a symlink or alternate command name, that command basename is used as the image key.

The image key must exist in the internal `IMAGES` map. Unsupported image keys are fatal errors.

Current image map:

```text
ubuntu -> https://cloud-images.ubuntu.com/resolute/current/resolute-server-cloudimg-amd64.img
```

## Commands

### `there` or `there auto`

Ensures the project-local instance exists and is running, then replaces the wrapper process with an interactive Lima
shell in `/here`.

If the instance already exists and is running, `there` enters it directly.

If the instance exists but is not running, `there` force-stops any stale state, starts the instance, then enters it.

If the instance does not exist, `there` creates it with:

- `--containerd none`
- `--tty=false`
- one writable mount from the host physical current directory to guest `/here`
- `.firmware.legacyBIOS = true` only when the host is detected as Linux booted without EFI
- the selected image URL

`--tty=false` disables Lima's interactive first-run configuration menu and proceeds with the generated configuration.

## Host Firmware Detection

`there` avoids forcing Lima's legacy BIOS mode on hosts where that setting is not needed or cannot be determined safely.

The detection rules are:

- macOS hosts do not use `legacyBIOS`.
- Linux hosts with `/sys/firmware/efi` do not use `legacyBIOS`.
- Linux hosts with `/sys/firmware` but without `/sys/firmware/efi` use `legacyBIOS`.
- Other hosts, or hosts without an inspectable `/sys/firmware`, do not use `legacyBIOS`.

This keeps the compatibility workaround scoped to Linux hosts that appear to have been booted through legacy BIOS. The
script does not depend on `bootctl`, `efibootmgr`, or other optional EFI tooling for this decision.

Tests may override the firmware sysfs root with `THERE_SYS_FIRMWARE`. Normal execution should leave it unset.

### `there start`

Starts an existing project-local instance, then replaces the wrapper process with an interactive Lima shell in `/here`.

If the instance does not exist, this command warns and exits non-zero. It does not create a new instance.

If the instance is already running, it enters the shell directly.

### `there run COMMAND [ARG...]`

Ensures the project-local instance exists and is running, then runs the provided command inside the guest with `/here` as
the working directory.

Calling `there run` without a command is a fatal usage error.

### `there stop`

Stops the existing project-local instance.

If the instance does not exist, this command warns and returns successfully.

If the instance exists but is not running, this command warns and returns successfully.

### `there destroy`

Deletes the existing project-local instance.

If the instance exists, `there destroy` first attempts to stop it and ignores stop failure, then deletes it with
`limactl --tty=false delete`.

If the instance does not exist, this command warns and returns successfully.

### `there prune`

Runs `limactl prune`. This command is not scoped to the project-local instance by name; it delegates pruning behavior to
Lima.

### `there help`, `there --help`, or `there -h`

Prints a compact AI-friendly and human-readable command reference suitable for pasting into repository agent
instructions. The output documents:

- the project-local VM model
- the `/here` mount contract
- instance naming shape
- image selection by command basename
- supported commands
- create/start/idempotence behavior
- host firmware handling
- short examples

Unknown commands print usage and exit with status `2`.

## Dependencies

`there` requires:

- Bash
- `limactl`
- `grep`
- `awk`
- `cut`
- either `md5sum` or `md5`

If neither `md5sum` nor `md5` is available, `there` exits with a fatal error.

## Tests

Run the shell test suite with:

```bash
tests/there
```

The tests fake `limactl` and `uname`, so they do not create or start real Lima instances.

## Error And Warning Behavior

Fatal errors are written to standard error with an `E:` prefix and exit with status `1`.

Recoverable warnings are written to standard error with a `W:` prefix.

Tracing is enabled when the `TRACE` environment variable is non-empty.

## Invariants

- Instance identity is a function of the physical current directory and selected image key.
- The host project directory is always exposed to the guest as writable `/here`.
- Lima legacy BIOS mode is opt-in by host detection, not a global default.
- `auto` and `run` may create a missing instance.
- `start` only starts or enters an existing instance.
- `stop` and `destroy` are idempotent for missing instances from the caller's perspective.
- `prune` is global Lima maintenance, not per-instance maintenance.
