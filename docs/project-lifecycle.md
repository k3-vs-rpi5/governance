[中文](project-lifecycle.zh-CN.md)

# Project lifecycle

## Entry gate

A project begins only after the shared OS/runtime/compiler foundation passes
and the lower-level dependencies are stable. The project card declares owner,
upstream source and version, dependency layer, supported tracks, correctness
oracle, primary metric and direction, result modes, CPU sets, telemetry, raw
schema, risks, and stop criteria.

Use code-shipped tests first. Use official suites for standardized benchmarks
such as CoreMark. Add a local test only when deleting it would leave an enduring
repository-owned contract unprotected; use the smallest deterministic case at
the lowest authoritative layer and do not duplicate upstream coverage.

## Preregistered matrix

Before the first performance run, freeze:

1. `rpi-native`, `k3-compatible`, and `k3-development` runtime identities;
2. `default-parity` and `best-achievable` rules;
3. source commit, patch manifest, compiler/linker/libc, flags, and CPU sets;
4. warmups, sessions, samples, minimum duration, seeds, and acceptance statistics;
5. frequency, temperature, memory, bandwidth, throttling, swap, perf, and I/O telemetry;
6. official correctness command, external evidence-root format, checksums, and
   the reports-repository identity.

## Verify-to-land cycle

Every candidate follows the same complete cycle:

1. reproduce the locked baseline and verify its identity;
2. locate a hotspot with profiler, counter, and assembly/compiler evidence;
3. state one falsifiable mechanism and the expected affected metric;
4. make the smallest general change and run official correctness tests;
5. screen with a quick run and promote only through the predefined gate;
6. run formal repeated measurements across required tracks and controls;
7. either land an upstream-ready patch/lock or record a rejection with raw-run IDs;
8. independently reproduce using only the public build/run scripts and documents.

The cycle is not complete when a score merely improves. It is complete when the
mechanism, correctness, statistical result, environmental identity, patch,
reproduction, and report all agree.

## Review and solidification

The project owner reviews source integrity, patch order and zero-fuzz replay,
compiler versions, runtime digests, CRC or upstream correctness results, sample
quality, telemetry, baseline/optimized separation, limitations, and both
language reports. Required fields may be `blocked` only for a technically
non-applicable track, never because a temporarily unavailable device was skipped.

After project basic validation passes, the independent project repository may
receive its first clean commit. A report status changes only in the separate
reports repository after formal evidence and human review. The first push still
waits for a clean release-manifest synchronization and explicit remote approval.

## Upstream handoff

Accepted optimization patches are reordered and described according to the
target upstream's contribution format. Each patch must be minimal, general,
bisectable, tested independently, free of benchmark-only result fabrication,
and accompanied by the exact reproduction evidence. Compiler improvements land
in the compiler project when that is the correct owner; workload-source
diagnostics never masquerade as an official-score patch.
