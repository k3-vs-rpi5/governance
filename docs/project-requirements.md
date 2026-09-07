[中文](project-requirements.zh-CN.md)

# Project requirements

## Project repository model

Every admitted benchmark is an independent Git repository selected by the
Google `repo` manifest. CoreMark is project ID `coremark`; there is no umbrella
`cpu-foundation` project. Shared device, runtime, environment, telemetry, and
evidence validation belongs to foundation and must not be copied into a project.
Formal results are recognized only from a revision-pinned complete workspace.

## Required three-track matrix

Every optimization or validation project runs, or technically marks
non-applicable, all three tracks: `rpi-native`, `k3-compatible`, and
`k3-development`. Reports never merge the two K3 environments. Raspberry Pi is
the script-validation origin; K3 Bianbu is the primary development environment.

## Required two-mode results

Every project publishes `default-parity` and `best-achievable` separately.
Default parity uses the same official source, port, parameters, seeds, duration,
CPU set, telemetry contract, and optimization class across all tracks. Best
achievable may use reviewed K3 ISA/compiler/runtime/patch improvements. The best
result never replaces the default result.

## Complete baseline information

Every result report records:

1. board/CPU model, architecture, topology, online cores, CPU set, and visible memory;
2. configured frequency, average sampled frequency, sampling interval/method, and governor;
3. average temperature, sensor paths, sampling method, throttling, swap, and active services;
4. OS release, kernel, loader, libc, compiler, linker, full flags, and executable digests;
5. runtime-profile digest, rootfs/sysroot/wrapper identity, and ABI compatibility outcome;
6. memory-bandwidth suite/method/result plus cache, branch, perf, and auxiliary counters;
7. actual UFS/eMMC/SD storage class, cache policy, I/O exclusion, wired network, and Type-C supply control;
8. official suite/version/source/checksum, commands, warmups, sessions, samples, and statistics;
9. raw-run IDs, validation outcome, deviations, limitations, and reviewer.

Unknown values are `not-recorded` with a reason. Reports provide a complete
English Markdown and a complete Chinese Markdown; a translated heading without
translated body content is noncompliant. Logs use UTC names
`YYYYMMDDTHHMMSSZ-<track>-<suite>-<stage>.log` and raw runs are append-only.

## Stable report policy

The separate reports repository owns one stable directory per project:
`<project-id>/`. Repeated tests create a new append-only candidate beneath the
operator-selected evidence root and never update a report directly. After human
review, accepted compact evidence and its index are promoted to the reports
repository and the stable report is updated in place. Date-named report
directories, cycle subdirectories, and multiple current reports are forbidden;
Git history is the inheritance record.

## Fixed external delivery

Every public project delivers exactly:

1. `scripts/build.sh`, the only public build entry in the project repository;
2. `scripts/run.sh`, the only public run/test entry in the project repository;
3. `patches/` with bilingual notes, ordered patches, and `manifest.yaml`;
4. a project README pair containing complete reproduction instructions; and
5. `<project-id>/report.md`, `report.zh-CN.md`, `report.yaml`, and
   `evidence/index.yaml` in the separate reports repository.

The build entry fetches the pinned official/upstream source, verifies integrity,
installs missing declared packages through APT, verifies the exact compiler
chain, applies the manifest series with zero fuzz, and writes a build manifest.
It clears `PYTHONPATH`/`PYTHONHOME`, disables the user site, and records APT
ownership/path/version/SHA256 for Python build modules. A temporary compiler
APT repository redirects both `sourcelist` and `sourceparts` without modifying
system sources.

The run entry revalidates build/binary identity, executes the official or
shipped correctness suite first, runs the preregistered matrix, captures
telemetry, and seals raw output beneath an explicit local evidence root outside
all managed Git repositories. Quick mode is non-reportable; formal mode is the
only possible input to an accepted report, and report promotion remains a
separate human-reviewed operation.

## Code necessity gate

Before creating any source file, module, script, helper, test, fixture, or
framework, the author must stop and ask: **Is this code necessary?** The default
answer is no. Code may be added only when a one-sentence necessity rationale
shows that it:

1. is directly required by an approved public artifact or an enduring contract;
2. is not already covered by repository code, an official/upstream tool or test,
   configuration, documentation review, or preserved evidence;
3. is the smallest durable implementation and does not increase total complexity
   more than the risk it removes; and
4. has a deterministic verification method, a clear owner, and a deletion
   condition.

If the answer is uncertain, do not create the code. Prefer reuse, deletion,
configuration, documentation, official suites, or a simpler existing path.
Tests are code: a plan item, test count, coverage target, symmetry, hypothetical
future reuse, or a one-time migration is not sufficient justification. Record
one necessity rationale per new code file or cohesive test group in the working
plan or review record; do not scatter ceremonial comments through the code.
Code that no longer passes this gate is deleted together with the behavior it
alone protected.

## Test-source principle

Use the code's shipped tests whenever available. Standardized scores such as
CoreMark use the official suite, not a locally invented substitute. A missing
or inadequate upstream test may be supplemented only by a reviewed test with a
documented oracle, deterministic fixtures, boundary/negative/adversarial cases,
failure-path coverage, and independent reproduction. The report clearly labels
the supplemental result and never presents it as an official score.

## Local test decision rule

Add a local automated test only for an enduring, deterministic repository-owned
contract or regression: public interfaces and schemas, security or fail-closed
boundaries, subtle parsing/statistics/identity behavior, repository integration
or isolation, and hardware-independent acceptance gates. Use one minimal test at
the lowest authoritative layer and parameterize equivalent cases.

Do not duplicate official or upstream tests. Do not create unit tests for prose,
one-time migrations, static hardware facts, live readings, trivial declarative
values already schema-checked, or an invariant already covered at a lower layer.
Those cases belong in documentation review, the conformance ledger, qualification
evidence, official suites, or the stable report. Test quantity and coverage
percentage are not delivery goals. The complete decision and deletion rules are
normative in the pair `docs/repository-contract.md` and
`docs/repository-contract.zh-CN.md`.

## CPU and supporting suites

Memory bandwidth follows upstream STREAM; cache, syscall, thread,
cryptography, compression, allocator, and library tests use their upstream or
shipped suites. Every result states workload, metric, direction, unit, CPU set,
method, limitations, and publication terms. Free suites establish the initial
CPU baseline; licensed SPEC CPU content remains local and is not required for
the first free-suite baseline.

## CoreMark official-score lane

CoreMark 1.0 is pinned to
`b56889a7bd9d5624fbeff2a73ba69878e6a9c146`, must pass `make check`, and keeps
all `coremark.md5`-controlled workload files unchanged. Only `core_portme*`,
compiler/build/link options, and ISA selection may change for an official score.
Upstream main `1f483d5b8316753a742cbf5590caf5bd0a4e4777` is excluded because the
untouched checkout fails its own integrity check. Workload-source experiments
are labelled `diagnostic-only`.

Reportable tracks use the same `formal_runner_sha256` and platform-state
collector identity. Each raw package contains the scripts, compiler/wrapper
digests, telemetry, and `SHA256SUMS`. The K3 research target is
`7.7+ CoreMark/MHz` using measured average frequency. A quick candidate must
beat the locked best by at least `1%` before entering `3 sessions x 30 samples`.

## Compatible K3 compiler and runtime

The primary `k3-compatible` parity result runs compiler and runtime from the
locked Debian 13 riscv64 rootfs. GCC 14, binutils, glibc, loader, and package
versions match Raspberry Pi Debian 13 wherever both architectures publish the
same versions. A Bianbu high-version compiler against the low sysroot is a
separately labelled supplemental result and is never called equalized.

The wrapper or explicit `--sysroot`, portable ISA flags, loader/library paths,
package locks, and required ABI symbols are verified. Host-library leakage,
undeclared compiler fallback, a newer required glibc symbol, or a missing
rootfs/compiler digest fails closed.
