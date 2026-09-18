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
  toolchain `spacemit-toolchain-linux-glibc-x86_64-v1.2.4/`, for example
  `bin/riscv64-unknown-linux-gnu-gcc` (GCC 15.2.0) with its own `sysroot/`.
  This is the default for the K3 development and optimization path; a host
  compiler, a second cross toolchain, or an undeclared vendor package is not
  substituted for it. Compiler, linker, binutils, libc, and sysroot versions
  plus the compiler executable SHA-256 enter the build manifest and sealed
  evidence. A track whose environment lock names another compiler and runtime
  keeps that locked identity, and the difference is a declared deviation, never
  an undeclared substitution. A reviewed best-achievable lane may name another
  compiler when a project lock, a measured comparison, and human approval
  agree: the K3 `k3-development` best-achievable lane uses Clang 23 with the
  upstream SpacemiT X100 scheduling model and a board-generated LLVM profile,
  records the compiler, runtime, and profile digests in its evidence, and never
  enters a `default-parity` result.
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

## Core clusters: X100 and A100

The K3 exposes two clusters with different rules, and every artefact, test, and
report states which one it targets.

- X100 (cpu0-7) is an ordinary Linux cluster. Its code is compiled for the X100
  ISA, scheduled normally, attributed per process, and may be pinned with
  `taskset`.
- A100 (cpu8-15) cannot be entered by an ordinary task. `sched_setaffinity` and
  `taskset` reject every mask that contains an AI core - measured `EINVAL` for
  `8`, `0,8`, `1,9`, `8-15`, `0,8-15`, `1-8`, `4-11` and `0-15`, while every
  subset of 0-7 succeeds. A100 code is reached in two steps and only those two:
  1. compile the kernel path for the AI core ISA, which means enabling the IME2
     instruction family through `-march=..._xsmtvdotii` or
     `-mcpu=spacemit-a100`; a build that omits it silently contains no A100
     path;
  2. submit the work through a documented runtime that owns the compute cores:
     the Spine-Runtime C++ API (`spert::Stream`, `StreamConfig::core_ids`,
     `Grid`, `Context`), which is the vendor's own programming model for the
     `spacemit-k3` backend (`CC Core {8..15}`, 384 KiB per-core shared buffer);
     the llama.cpp SpacemiT backend; or the ONNX SpacemiT execution provider.
     Those runtimes reach the driver on the caller's behalf - the llama.cpp
     backend by writing to `/proc/set_ai_thread`, the others internally - and
     the driver then executes the work on the AI cluster.
- The A100 development path is therefore Spine-Runtime, not a private interface:
  build against the published `libspert` headers and pkg-config file, query the
  granted cores and shared-buffer size through `backend_info()` instead of
  hard-coding them, and request cores through `StreamConfig` rather than trying
  to pin a thread. `spacemit-k3-x100` is the vendor's compatible execution mode
  on the X100 core set, and `generic/qemu` is for functional validation only -
  neither reports the performance of the A100 cluster.
- A build manifest and every sealed evidence package record the flag that
  enabled the A100 path and the runtime that registers AI threads, or state that
  the artefact is X100-only.
- A claim that a workload used the A100 cluster must show engagement from the
  vendor side: the runtime's registration of the work and the TCM block
  occupancy that names the workload's pid (`spacemit-tcm-smi`), read together
  with the throughput arithmetic the workload's own size and token rate imply.
  Measured on the K3 on 2026-09-18, the Linux-side instruments cannot see this
  cluster at all: while a dense run produced 124 t/s of prefill, the kernel's
  own perf counted 1,633,901 cycles in two seconds across cpu8-15 against
  773,030,024 on cpu0, and `/proc/stat` credited cpu8 and cpu15 with zero user
  and system jiffies (401 and 402 idle jiffies of 402). Per-process CPU time is
  never presented as A100 work, because the AI cores have no per-process
  accounting, and per-CPU counters or `/proc/stat` are never presented as A100
  evidence, because on this board they do not see it.
- Path selection follows the hardware, not the build host: the arch id in
  `/proc/cpuinfo` decides (`0x5064` for X100, `0xA064` for A100), so one binary
  may carry both kernel sets and still run correctly on a board without an AI
  cluster.
- Cluster measurements are not interchangeable. Frequency, cache hierarchy and
  the measured memory roof differ per cluster, and a comparison across clusters
  states both ceilings.
- Vendor AI runtimes are used through documented interfaces: the llama.cpp
  SpacemiT backend, the Spine-Runtime API, the ONNX SpacemiT execution provider,
  and its published plugin API and debug-profile options. The vendor stack is
  pinned by identity - the board's llama.cpp build `5a23f07a4` is vendor tag
  `v0.1.7-a0` (commit `5a23f07a4519`), the Spine-Runtime SDK release `0.6.2`
  carries `libspert.so.0.6.2`, and the ONNX package `2.0.6` records the commit
  it was built from - and a closed component's released package identity enters
  the evidence.

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
