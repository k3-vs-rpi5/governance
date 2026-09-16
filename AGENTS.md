[中文](AGENTS.zh-CN.md)

# AI/Human Collaboration Contract

This file is the workspace-level operating contract copied from the governance
repository by the Google `repo` manifest. Read it before changing any component.
A project-local `AGENTS.md` may impose stricter rules but cannot relax this file.

## Start-of-work checks

1. Read the selected release/development manifest, governance documents, the
   affected project manifest, and the latest local evidence state.
2. Before creating code or a test, ask: **Is it necessary, and which declared
   contract would break if it were deleted?** If there is no precise answer, do
   not create it.
3. Inspect the current Git state and preserve unrelated or uncommitted work.
4. Never read, print, commit, or upload credential values. Private device data,
   caches, build outputs, and local candidate evidence stay outside Git.
5. Local device access records, if present, are workspace-root files named
   `.ssh-k3` and `.ssh-rpi`. They are local-only credential files for the K3 and
   Raspberry Pi devices, may contain address, account, and authentication
   material, must be mode `0600`, and must never be read, printed, committed, or
   uploaded by AI contributors. AI contributors may check only their existence,
   path, ownership, and permission bits unless a human explicitly operates the
   credential material.

## Development rules

- Establish correctness, environment, measurement, and evidence contracts before
  optimization.
- Use official or upstream tests and benchmark suites first. Supplemental tests
  protect only enduring repository-owned gaps and are never presented as an
  official score.
- Work from lower dependencies upward. Every optimization follows hypothesis,
  one-variable experiment, evidence, conclusion, and accepted/rejected patch.
- Record exact source, compiler, linker, libc, runtime, flags, patch, CPU,
  frequency, temperature, memory, storage, network, and relevant service state.
- Preserve the three tracks `rpi-native`, `k3-compatible`, and
  `k3-development`, plus separate `default-parity` and `best-achievable`
  results. Disclose every material mismatch.
- Each project exposes only `scripts/build.sh` and `scripts/run.sh`. Reports are
  never written as a side effect of a benchmark run.
- Device runs use host-to-device code delivery. Push the selected workspace or
  sealed build inputs from the host to the boards, then execute the project
  scripts on the boards. Do not change the benchmark method by having the boards
  clone, pull, download, or otherwise select code independently.
- Prepare accepted source changes as reproducible, zero-fuzz, upstream-grade
  patches.

## Workspace build, device, and optimization workflow

- Every cross build of a binary that runs on a board uses the workspace
  toolchain `spacemit-toolchain-linux-glibc-x86_64-v1.2.2/`, for example
  `bin/riscv64-unknown-linux-gnu-gcc` (GCC 15.2.0) with its own `sysroot/`.
  This is the default for the K3 development and optimization path; a host
  compiler, a second cross toolchain, or an undeclared vendor package is not
  substituted for it. Compiler, linker, binutils, libc, and sysroot versions
  plus the compiler executable SHA-256 enter the build manifest and sealed
  evidence. A track whose environment lock names another compiler and runtime
  keeps that locked identity, and the difference is a declared deviation, never
  an undeclared substitution.
- Device work reaches the selected board through the local-only workspace
  credentials `.ssh-k3` (K3) and `.ssh-rpi` (Raspberry Pi): the host pushes the
  cross-compiled result, executes it there, and collects and analyses the raw
  output, telemetry, and logs. Credential values stay unread outside an
  operator-controlled shell, and the board never selects its own code.
- A project keeps the pinned upstream source in `src/<upstream>` and its own
  automation in `src/<project>_project/`. The local checkout pins the exact
  commit and integrity check that a git download cannot guarantee, and the
  Python package solidifies retrieval, build, delivery, execution, telemetry,
  and evidence behind the two public scripts.
- Optimization is edited, built, and measured in that local checkout, and an
  accepted optimization is then frozen into the Git repository and recorded as
  an ordered, zero-fuzz, reversible patch with its manifest entry and bilingual
  note. A compile-parameter optimization with no landing place in the upstream
  checkout lands in `src/<project>_project/`; most optimizations belong in the
  source code rather than in compile parameters.

## Authority and review

AI contributors may inspect, implement an approved scope, run in-scope checks,
and prepare evidence and review material. Human approval is required for project
admission, architecture changes, accepted environment deviations, report
promotion, destructive cleanup, remote configuration, and the first push.

No contributor may claim completion without fresh verification evidence. A
failed or interrupted run is retained locally as `invalid` or `blocked`; it is
never silently promoted or substituted with a similar run.

## Durable handoff

Chat is not an authoritative store. Requirements, approvals, exceptions,
versions, evidence identities, and remaining blockers needed after a new
`repo sync` must be written to the appropriate tracked contract or sealed local
evidence. Do not add documents, scripts, frameworks, or tests solely to narrate
work that an existing canonical artifact already records.
