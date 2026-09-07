[中文](repository-contract.zh-CN.md)

# Multi-repository benchmark contract

Document ID: `repository-contract`
Content revision: `2`
Owner: governance maintainers
Review state: approved design

## Purpose and authority

This contract governs the K3 Pico versus Raspberry Pi 5 evaluation workspace.
Its purpose is trustworthy, publicly auditable performance comparison and
upstream-grade optimization, not production of attractive but uncontrolled
scores.

The Google `repo` release manifest selects the exact governance, foundation,
project, and optional reports revisions. The root `AGENTS.md`, this contract,
the affected project manifest, and the selected environment locks are mandatory
inputs to every development session. Chat history is not normative.

## Repository boundaries

The workspace root is not a Git repository. The manifest repository composes
independent Git repositories:

- governance owns policy and AI/human collaboration rules;
- foundation owns shared device, environment, telemetry, statistics, and
  evidence validation;
- each benchmark or optimization target owns one project repository;
- reports owns one stable public report and evidence index per project.

CoreMark is the first independent project, with ID `coremark`. There is no
`cpu-foundation` umbrella project. STREAM, lmbench, OpenSSL, allocators, and
other suites enter as separate projects only after dependency-order admission.

Reports are assigned `reports,notdefault`. Default code migration and normal
`repo sync` do not transfer reports. A release manifest may pin the reports
revision to preserve traceability without including its checkout by default.

## Code necessity

Before creating a source file, module, helper, script, fixture, test, or
framework, ask:

> Is this code necessary, and which declared contract would be broken if it
> were deleted?

No is the default. A new item is allowed only when a concise review rationale
identifies an enduring approved contract, shows no adequate upstream or existing
path, chooses the smallest durable implementation, gives deterministic
verification, and names its owner and deletion condition. A plan line, coverage
target, test count, file symmetry, hypothetical reuse, or one-time migration is
not a justification.

## Test policy

Official and code-shipped correctness tests are authoritative. Standard scores
such as CoreMark use the official suite and rules. A local test does not replace
or redefine an official score.

Local automated tests protect only enduring repository-owned behavior:

1. zero-fuzz patch identity, application, and reversal;
2. fail-closed environment and control-variable qualification;
3. reproducibility and identity closure of public build/run entries; or
4. traceability and integrity between accepted evidence and a report.

A shared contract is tested once in foundation. Projects do not duplicate it.
Tests for static prose, trivial wrappers, third-party internals, live hardware
facts, one-time migration, or a behavior already covered by upstream, a schema,
or a lower authoritative layer are removed. Equivalent cases are parameterized
in the smallest useful test. Test quantity and coverage percentage are never
release goals.

## Documentation policy

Every maintained project-owned Markdown document is a complete pair:

```text
name.md
name.zh-CN.md
```

The `.en.md` suffix, mixed title-only translations, orphan counterparts, broken
local links, and materially different commands, versions, hashes, metrics, or
thresholds are forbidden. Canonical documents are updated instead of creating
new overlapping guides. Reports follow their separate rule below.

## Project admission and dependency order

A project begins only after the shared OS/runtime/compiler foundation and every
affected lower dependency pass qualification. Its `project.yaml` declares the
owner, dependency layer, upstream source, applicable tracks, correctness oracle,
metrics, CPU sets, telemetry, result modes, risks, deviations, and stop criteria.

Validation always precedes optimization. Work proceeds from lower libraries and
runtimes to dependent libraries and applications. A project with an unstable
lower dependency is `blocked`, not silently promoted.

## Three-track environment contract

Every applicable project separates:

1. `rpi-native`: Raspberry Pi 5 4 GB, script-validation origin, pinned current
   OS and lower-version compiler/runtime;
2. `k3-compatible`: K3 Pico using a content-locked rootfs/chroot and the closest
   available matching compiler/runtime packages; and
3. `k3-development`: K3 Pico using native Bianbu and the pinned newer development
   compiler/runtime.

The same architecture-independent source, workload, inputs, benchmark
parameters, CPU-count class, affinity policy, warmups, sessions, sampling, and
telemetry policy apply to `default-parity`. Exact compiler, linker, libc,
binutils, runtime, flags, and executable identities are recorded for each
architecture. A material difference is a declared deviation; while unresolved,
the result is not called strictly aligned.

Missing declared packages are installed through pinned APT sources. A specified
upstream compiler may substitute only when its version, origin, digest, and
runtime boundary are locked. Host loader/library leakage and silent compiler
fallback fail closed.

Stable Type-C power and wired Ethernet are required on both devices. CPU timing
excludes source download, package installation, compilation, and storage setup.
The actual eMMC/UFS/SD class and cache policy are recorded; unavoidable I/O is
reported separately. One-core and four-core modes use explicit affinity and do
not require a custom image when the installed OS can enforce them reliably.

## Result modes and optimization cycle

Every project keeps `default-parity` and `best-achievable` separate. The best K3
result never replaces the matched default result.

An optimization cycle is complete only after:

```text
locked baseline
→ profiler/counter/assembly hotspot
→ falsifiable hypothesis
→ one-variable candidate
→ upstream correctness
→ quick screening
→ formal repeated validation
→ accepted or rejected disposition
→ upstream-grade patch
→ reviewed report update
```

Benchmark-source shortcuts, hidden workload changes, relaxed correctness, or
unrecorded environment changes invalidate a result. Generic improvements are
preferred. A legitimate platform or compiler change is allowed when disclosed,
owned by the correct upstream, and kept outside official workload sources unless
the benchmark rules explicitly permit it.

## Project public interface

Each project repository exposes exactly two Shell entry points:

```text
scripts/build.sh
scripts/run.sh
```

`build.sh` retrieves and verifies locked upstream source, records the complete
compiler chain, installs missing declared dependencies through APT, applies the
ordered patch manifest without fuzz, runs upstream correctness, and seals build
identity. It fails on dirty source, missing locks, unverified packages, patch
fuzz, or undeclared fallback.

`run.sh` accepts a sealed build and an explicit absolute local evidence root.
It revalidates identities, qualifies the selected environment, runs official
correctness before performance, performs declared warmups and samples, records
telemetry, and seals an append-only candidate. It never changes reports or Git.

Additional Shell helpers are forbidden. Non-Shell project logic is added only
when it passes the code-necessity gate. Reproduction instructions belong in the
project README pair.

## Measurement and evidence

Every formal candidate records board/CPU identity, architecture and topology,
online cores and affinity, visible memory, configured and sampled frequency,
average temperature, governor, throttling, swap, services, OS, kernel, loader,
libc, compiler, linker, flags, executable digests, runtime-profile digest,
rootfs/sysroot identity, memory-bandwidth context, storage, wired network,
official suite/source/checksum, commands, warmups, sessions, samples, statistics,
deviations, and reviewer state.

Unknown mandatory information is `not-recorded` with a reason and prevents
formal promotion when it affects interpretation. Failed or interrupted runs are
retained locally as `invalid` or `blocked`. Device reboot, replacement, thermal
or frequency instability, missing telemetry, runtime drift, checksum failure,
or official correctness failure requires requalification.

## Report policy

The separate reports repository contains one stable logical report per project:

```text
<project-id>/report.md
<project-id>/report.zh-CN.md
<project-id>/report.yaml
<project-id>/evidence/index.yaml
```

No date, cycle, `latest`, `related`, or revision report directories are allowed.
The report pair is updated in place and Git history is its inheritance trail.
`report.yaml` pins manifest, governance, foundation, project, source, compiler,
patch, and evidence identities. Accepted compact evidence is immutable and
content-addressed. External large logs require a stable locator, SHA-256, byte
size, and availability state.

Only a human-reviewed promotion may update reports. Project execution cannot do
so. Missing evidence cannot be substituted with a similarly named run; the
claim remains a limitation or `blocked-retest` until reproduced.

## Git and migration policy

The official migration method is `repo init` with an approved manifest followed
by `repo sync`. Direct workspace copying is not official because it may include
ignored secrets and uncontrolled state. Root collaboration files are copied
from governance by the manifest.

Before splitting history, create and verify an ignored recovery bundle. Each
target repository receives its first clean commit only after its source tree
passes basic validation. Existing monorepo history remains in the recovery
bundle and is not injected into new component histories.

No network remote or push is configured implicitly. The exact remote URL,
branch, and refspec require human approval after a clean release-manifest
synchronization. Credentials, private inventory, toolchain downloads, caches,
builds, local candidates, and recovery material never enter a managed Git
repository.

## Completion gates

A component is ready for its first clean commit only when its applicable
documentation, syntax, upstream, patch, package, and focused contract checks
pass. A project is ready for a public result only after formal three-track
evidence, default/best separation, environment deviation review, independent
reproduction, bilingual report review, and evidence-identity closure pass.

The complete workspace is ready for a first push only when an immutable release
manifest reconstructs the same code revisions in a clean workspace, default
synchronization omits reports, explicit report synchronization obtains the
pinned report, root collaboration rules are present, tracked content contains no
secret, and the human owner approves the remote operation.
