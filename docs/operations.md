[中文](operations.zh-CN.md)

# Laboratory operations

## Local preparation

Use a dedicated virtual environment. Generic dependencies absent from a target
are installed through APT and their exact package versions are recorded. Do not
silently use pip, the user site, a host compiler, or an undeclared vendor
repository to satisfy a reproducibility requirement.

```bash
python3 -m venv foundation/.venv
foundation/.venv/bin/python -m pip install -e 'foundation[dev]'
foundation/.venv/bin/python -m pytest foundation/tests
PYTHONPATH=foundation/src foundation/.venv/bin/python -m labctl.cli docs-check \
  --root governance --catalog governance/config/documentation.yaml
```

Device addresses, accounts, passwords, and optional identity-file paths are
stored only in `.device-credentials`. The file is ignored by Git, must be mode
`0600`, is never printed, and is sourced only by an operator-controlled shell.

## Installed OS and device qualification

Raspberry Pi 5 and K3 Pico use stable Type-C power and wired Ethernet. This
baseline does not build or distribute a custom flash image. The installed
Debian and Bianbu systems are locked by the environment manifest, runtime
profile, package/toolchain hashes, service and mount digests, and reboot
evidence. After any flash, OS upgrade, or board replacement, collect a new SSH
probe and repeat those gates before accepting results.

Do not rebuild an image merely to limit cores. If the OS exposes the required
cores reliably, bind the process and interrupts to a declared one-core or
four-core CPU set. A replacement K3 board receives a new device identity and
qualification record but reuses the logical `k3-compatible` and
`k3-development` profiles after requalification.

## Runtime equalization

The three runtime tracks are immutable inputs:

1. `rpi-native` uses the Raspberry Pi Debian baseline and its APT compiler;
2. `k3-compatible` uses the locked Debian riscv64 rootfs, compiler, loader, and libraries;
3. `k3-development` uses the Bianbu native runtime and declared newer compiler.

For strict parity, compiler and runtime both come from the compatible rootfs.
A Bianbu high-version compiler with the low sysroot is supplemental only. Check
ELF architecture, interpreter, required symbol versions, sysroot paths, compiler
executable digest, linker, libc, and wrapper digest before execution. Host
library leakage or a missing package lock fails closed.

Temporary local APT repositories must redirect both `sourcelist` and
`sourceparts`; they must not overwrite system source configuration or consume
existing third-party sources implicitly. Clear `PYTHONPATH` and `PYTHONHOME`,
disable the user site, and record the imported module path and digest for Python
build dependencies.

## Timed-run preparation

Before every formal project run:

1. verify source, patch manifest, compiler chain, runtime profile, and binary digests;
2. verify wired networking, CPU set, governor, throttling state, swap, and active services;
3. sample configured/actual frequency, temperature, visible memory, and storage state;
4. run the official or shipped correctness suite before performance samples;
5. keep source download, package installation, compilation, and storage I/O outside timing;
6. seal logs, telemetry, manifests, copied scripts, and checksums beneath the
   explicit local evidence root outside every managed Git repository.

The current K3 UFS 2.2 device and Raspberry Pi SD card are not treated as equivalent storage.
Datasets are prepared before timing, caches are either deliberately warmed or
dropped under one declared policy, and any unavoidable I/O is reported rather
than included in a CPU claim.

Default parity does not selectively stop services or isolate K3 background work
onto spare cores when the four-core Raspberry Pi cannot receive the same
control. Instead, each track locks the active-service and mount digests, records
interrupt and swap activity, and must pass the same preregistered idle-stability
gate before every session. Package managers, update jobs, and other known noisy
work must be inactive. A session that fails the idle gate is invalid and rerun;
the threshold and observation window belong in the project manifest before
measurement. K3-only service or cpuset isolation is allowed only in a separately
labelled `best-achievable` result, never in `default-parity`.

## Project execution

Each public project has exactly two Shell entry points:

```bash
projects/coremark/scripts/build.sh --help
projects/coremark/scripts/run.sh --help
```

Quick mode is diagnostic and never reportable. Formal mode must use the locked
matrix, sufficient duration, predefined warmups/sessions/samples, full telemetry,
and append-only raw output. A failure is recorded as failed or blocked; operators
must not edit a manifest to convert it to passed.

### Remote long-running jobs

An SSH foreground process is not an evidence-safe executor for a formal run.
Start every long-running device job through a host-level supervised service that
survives SSH disconnects, runs as the declared non-root device account, and
retains `Result`, `ExecMainCode`, `ExecMainStatus`, and journal records. A user
service is acceptable only when that account has verified linger; otherwise use
a system transient service with an explicit `User` and `WorkingDirectory`.

Record the boot ID before launch and after completion. Acceptance requires the
same boot ID, a successful supervisor result and exit status, the exact expected
sample counts, the project validation gates, and a valid checksum closure. An
SSH disconnect alone does not invalidate a supervised run. A device reboot,
supervisor stop, missing sample, or incomplete manifest always invalidates it;
preserve the failed evidence locally and repeat device qualification before the
next attempt. Supervisor use changes delivery reliability only and must not
change benchmark affinity, priority, environment, or timing policy.

## Verification and release

Run only the local contract, Shell syntax, ShellCheck, package-build, upstream,
device, formal, and clean-workspace gates applicable to the component. Verify
the release manifest from a new workspace:

```bash
repo init -u <approved-manifest-url> -m releases/coremark-baseline.xml
repo sync
test ! -e reports
```

The angle-bracket value above is an operator-supplied approved URL, not tracked
manifest content. Reports require an explicit reports-group synchronization.
Never push automatically. The exact remote URL, target branch, and refspec need
explicit approval after the clean-workspace result is reviewed.
