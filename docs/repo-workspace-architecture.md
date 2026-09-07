[中文](repo-workspace-architecture.zh-CN.md)

# Repo Workspace Architecture

Document ID: `repo-workspace-architecture`
Content revision: `1`
Owner: repository maintainers
Review state: approved design candidate

## Purpose

This document defines the approved target architecture for migrating the whole
K3-versus-Raspberry-Pi-5 benchmark workspace to Google `repo` management. It
separates governance, shared validation infrastructure, benchmark projects, and
reports into independent Git repositories while preserving one reproducible
workspace and the established AI/human collaboration model.

This design supersedes any current rule that treats CoreMark as a cycle inside
`cpu-foundation`, stores reports in the code repository, or treats the current
monorepo root as the final Git ownership boundary. Those documents are migration
inputs until the implementation plan replaces their conflicting sections.

## Scope and non-goals

The migration unit is the complete `k3Vrp5` workspace reconstructed from a
pinned manifest. CoreMark is the first independent benchmark and optimization
project. Future suites such as STREAM or lmbench become separate projects only
after their own admission review.

The design does not:

- copy credentials, caches, build outputs, local candidates, or unreviewed raw
  runs;
- carry reports in the default code migration;
- create one project merely to group unrelated CPU suites;
- add test-count or coverage-percentage targets; or
- authorize a public performance claim before formal evidence review.

## Repository topology

The workspace root is managed by Google `repo` and is not itself a Git
repository.

```text
k3Vrp5/
├── .repo/                      repo metadata and selected manifest
├── AGENTS.md                   copied from governance by the manifest
├── governance/                 policy and collaboration Git repository
├── foundation/                 shared validation Git repository
├── projects/
│   └── coremark/               independent CoreMark Git repository
└── reports/                    optional, non-default reports Git repository
```

The manifest repository is the version-set authority. It defines repository
URLs, paths, groups, and revisions. Development manifests may follow reviewed
branches. Every reproducible or public baseline uses a release manifest whose
project revisions are immutable Git commit IDs.

The `reports` project is assigned to the `reports` and `notdefault` groups.
Normal initialization and synchronization exclude it. A reviewer explicitly
selects the reports group when report access is required. A release manifest
may pin the reports commit so that code and report identities remain
cross-checkable without transferring report contents during normal migration.

## Repository responsibilities

### Manifest repository

The manifest repository owns only workspace composition:

- project names, remotes, paths, groups, and revisions;
- development and immutable release manifests;
- the governance `copyfile` rule that materializes the root `AGENTS.md`; and
- version-set metadata required to reproduce a synchronized workspace.

It does not own benchmark logic, environment checks, reports, or duplicated
policy prose.

### Governance repository

The governance repository owns complete English and Chinese policy pairs,
project admission rules, the AI/human collaboration contract, directory and
documentation rules, and review checklists. Its authoritative `AGENTS.md` is
copied to the workspace root by `repo`, so a newly synchronized workspace
restores the collaboration mode without relying on chat history or machine-local
memory.

### Foundation repository

The foundation repository owns shared device qualification, environment
profiles, three-track comparison contracts, telemetry and evidence schemas,
common validators, and the smallest justified set of repository-owned tests.
It does not own benchmark-specific patches, score parsing, or reports.

### Project repositories

Each admitted benchmark or optimization target owns one independent Git
repository. The first is `projects/coremark`. A project owns its source and
toolchain locks, its two public entry points, benchmark-specific adapters,
accepted patches, reproduction instructions, and only the minimal tests needed
for its enduring project-specific contracts.

Repository independence means independent ownership and Git history. Formal
results are recognized only from the complete workspace selected by a release
manifest; an arbitrary standalone project clone is not a controlled formal
environment.

### Reports repository

The reports repository owns the reviewed public conclusion and public evidence
index for each admitted project. It contains no benchmark runner or duplicated
foundation validator. It is maintained and migrated separately from code.

## AI/human collaboration contract

The collaboration model is repository state, not conversation state. The root
`AGENTS.md` defines the common authority boundary and workflow. A project may
provide a narrower project-local `AGENTS.md`, but it cannot relax governance,
validation, security, or publication requirements.

At the start of work, an AI or human contributor reads the selected manifest,
the governance contract, the project manifest, and the current evidence state.
AI contributors may inspect, implement approved changes, run in-scope
verification, and prepare review material. Human approval remains required for
project admission, architectural changes, accepted environment deviations,
promotion into the reports repository, destructive cleanup, remote selection,
and the first push.

Decisions needed after a new synchronization are recorded in tracked contracts,
manifests, and review records. Measurement detail is recorded in local sealed
evidence. Chat history is never the sole record of a requirement, approval,
exception, result, or next action. Neither AI nor human review may waive an
official correctness or evidence gate without changing the governing contract
through the same reviewed design process.

## One project, one report

Each project has one stable logical report. English and Chinese files are two
complete language representations of that same report.

```text
reports/
└── coremark/
    ├── report.md
    ├── report.zh-CN.md
    ├── report.yaml
    └── evidence/
        ├── index.yaml
        └── <content-digest>/
```

The report files are updated in place. Dates, `latest`, `final-v2`, and
`related` directories or parallel current reports are forbidden. Git history is
the inheritance and audit trail. `report.yaml` records the exact manifest,
project, foundation, source, toolchain, patch, and evidence identities used by
the current conclusion.

Previously published compact evidence is immutable. New accepted evidence uses
a new content-addressed directory. Large complete logs may live in an external
archive, but the evidence index must record a stable locator, SHA-256, byte
size, and availability state. Missing, unstable, or unqualified runs remain in
a local candidate area and never enter the reports repository.

## Three-track environment contract

Every project runs or explicitly marks as technically inapplicable these three
tracks:

| Track | Purpose | Required environment |
|---|---|---|
| `rpi-native` | Script-validation origin and comparable default result | Raspberry Pi 5 4 GB, pinned current OS, pinned lower-version compiler chain and runtime |
| `k3-compatible` | Strict controlled comparison with Raspberry Pi 5 | K3 Pico using a locked rootfs/chroot with matched software versions, source, parameters, and procedure |
| `k3-development` | Primary K3 development and best-achievable result | Native Bianbu with a pinned newer compiler chain and runtime, with every material version recorded |

Environment parity never claims that Arm64 and RISC-V use identical binaries.
Parity is established in this order:

1. identical upstream source revision, patch state, workload, and input data;
2. identical compiler release, linker, build options, and language runtime where
   both architectures provide them;
3. the same distribution suite, package versions, and library configuration as
   far as both architectures support them;
4. identical CPU-count class, affinity policy, iterations, warmups, telemetry,
   network policy, and background-service policy; and
5. explicit recording of kernel, libc, binutils, CPU frequency, temperature,
   memory, swap, storage, throttling, and all unavoidable differences.

An unresolved material difference is entered in the deviation record. The
corresponding result must not be described as strictly aligned.

The K3 compatible environment uses an isolated, content-locked rootfs/chroot
and does not replace the Bianbu host environment. Missing compiler-chain
packages are installed through pinned APT sources. If both architectures cannot
obtain the same package version, a specified upstream toolchain may be used only
when its source, version, download origin, and SHA-256 are locked and the
remaining runtime differences are disclosed.

## Required result modes

Every project preserves two distinct result modes:

- `default-parity` uses the same official source, workload, parameters, CPU
  policy, and allowed optimization class across the three tracks; and
- `best-achievable` records the best valid K3 result obtained through disclosed,
  technically legitimate compiler, runtime, ISA, or upstreamable source work.

The best-achievable result never replaces the default-parity result. A
platform-specific benchmark shortcut, hidden workload change, relaxed
correctness rule, or unrecorded environment change invalidates the result.

## Subproject development cycle

Every project follows this order:

```text
necessity review
→ dependency layer and project admission
→ upstream correctness tests
→ build/run script validation on Raspberry Pi 5
→ Raspberry Pi 5 default baseline
→ K3 compatible default baseline
→ K3 native default baseline
→ profiler, hotspot, and assembly analysis
→ one-variable candidate optimization
→ correctness and regression validation
→ upstream-grade patch preparation
→ formal three-track reproduction
→ reviewed reports-repository update
```

Each optimization forms one closed cycle: hypothesis, one-variable experiment,
evidence, conclusion, and accepted or rejected patch disposition. Work proceeds
from lower dependencies upward. A project does not optimize an upper layer while
an affected lower-layer correctness or environment contract remains
unqualified.

Raspberry Pi 5 is the validation origin for public script behavior. Bianbu is
the primary K3 development environment. Formal conclusions require all
applicable tracks, not only the environment that produced the best score.

## Project repository contract

A project exposes the following stable responsibilities without duplicating
foundation behavior:

```text
projects/<project-id>/
├── project.yaml
├── README.md
├── README.zh-CN.md
├── sources.lock.yaml
├── environments.lock.yaml
├── scripts/
│   ├── build.sh
│   └── run.sh
├── patches/
│   ├── README.md
│   ├── README.zh-CN.md
│   ├── manifest.yaml
│   └── *.patch
├── src/                         only justified project-specific logic
└── tests/                       only justified project-specific contract tests
```

`build.sh` is the only public build entry. It retrieves locked upstream source,
verifies integrity, obtains the declared compiler chain, applies the ordered
patch series without fuzz, runs upstream correctness checks, and seals build
identity.

`run.sh` is the only public execution entry. It consumes a sealed build, invokes
foundation qualification, runs official correctness validation before
performance sampling, performs declared warmups and repetitions, captures the
required telemetry, and seals candidate evidence. It never edits a report.

Additional Shell helpers are forbidden. A non-Shell helper is permitted only
when the necessity gate proves that the behavior cannot remain clear and
reliable in an existing component.

## Code and test necessity gate

Before creating any source file, module, script, helper, fixture, test, or
framework, the AI or human author must answer:

> Is this code necessary, and which declared contract would be broken if it
> were deleted?

No is the default. New code requires one concise rationale naming an enduring
contract, the absence of an adequate upstream or existing implementation, a
deterministic verification method, an owner, and a deletion condition.

Test quantity and coverage percentage are not goals. Upstream or official tests
are used first. A local automated test is allowed only to protect one of these
enduring repository-owned boundaries:

1. zero-fuzz patch application and reversal;
2. fail-closed rejection of an invalid control variable or environment;
3. reproducibility and identity closure of the two public entry points; or
4. traceability between accepted evidence and a report conclusion.

A shared boundary is tested once in foundation and not repeated in projects.
Tests for trivial wrappers, static prose, third-party implementation details,
one-time migration code, or behavior already covered by an official suite or a
lower authoritative layer are forbidden. Equivalent cases are parameterized in
the smallest useful test rather than expanded into additional files.

## Evidence flow and failure handling

Measurement output begins as local candidate evidence. A candidate is promoted
only after schema validation, environment qualification, upstream correctness,
checksum closure, and human review. Promotion copies the compact accepted
evidence into the reports repository and updates the single report pair plus
`report.yaml` in one reviewed change.

A failed or interrupted run is retained locally with an explicit `invalid` or
`blocked` state and reason. It cannot be silently discarded, converted into a
partial success, or selected by report tooling. Device replacement, reboot,
thermal instability, frequency mismatch, toolchain drift, host-library leakage,
missing telemetry, or official-test failure invalidates the affected formal
session and requires requalification before retry.

## Migration and Git history

The official migration mechanism is a fresh `repo init` against the selected
manifest followed by `repo sync`. Directly copying the old workspace is not an
official migration because it can carry ignored secrets, stale objects, and
uncontrolled results.

Migration proceeds in this order:

1. freeze the current worktree without pushing or deleting recoverable evidence;
2. classify every retained path as manifest, governance, foundation, project,
   report, local-only, or obsolete;
3. dissolve `cpu-foundation`, moving shared behavior to foundation and CoreMark
   behavior to `projects/coremark`;
4. move the unique reviewed report and accepted evidence to reports;
5. remove absolute paths and assumptions about the current monorepo root;
6. prepare candidate independent repositories and a local candidate manifest;
7. reproduce a fresh workspace with reports excluded by default;
8. verify CoreMark source acquisition, patch replay, build, quick run, evidence
   validation, and AI/human rule discovery; and
9. only then create the first clean commit in each repository.

The existing monorepo history is retained only in an ignored recovery bundle.
It is not injected into the new repository histories. No repository is pushed
until its basic validation passes and its exact remote URL, branch, and refspec
are approved. Final manifests never contain placeholder remotes.

Credentials and private device inventory live outside all managed Git
repositories. Caches, downloaded toolchains, build workspaces, local candidate
evidence, and recovery bundles are also excluded. The root collaboration file
is reproducibly generated from governance rather than maintained as an
untracked local convention.

## Acceptance criteria

The architecture is implemented only when all of the following are true:

1. default `repo` synchronization excludes reports;
2. explicitly selecting the reports group retrieves the pinned reports commit;
3. a fresh workspace receives the authoritative root `AGENTS.md`;
4. governance, foundation, CoreMark, reports, and manifest have independent Git
   histories and unambiguous ownership;
5. CoreMark exposes exactly one public build entry and one public run entry;
6. its locked patch series applies and reverses without fuzz against verified
   upstream source;
7. compiler, linker, libc, runtime, flags, and environment deviations are
   recorded for all applicable tracks;
8. `default-parity` and `best-achievable` remain separate;
9. every retained local test identifies one allowed enduring contract;
10. no secret, cache, temporary log, candidate result, or report is transferred
    by the default code migration;
11. the single CoreMark report pair traces every current claim to accepted
    evidence and exact repository revisions; and
12. an immutable release manifest reconstructs the same code version set in a
    clean workspace.

## Current-tree consequences

The current `docs/repository-contract*`, `docs/repository-layout*`,
`docs/project-requirements*`, and `docs/repository-consolidation-plan*` pairs must
be rewritten during implementation. In particular, the following old rules are
removed:

- CoreMark as a `cpu-foundation` cycle rather than an independent project;
- reports and raw-result ownership inside the code monorepo;
- one monorepo Git history and two-commit final shape; and
- project-local duplication of shared validation and report checks.

Rules that remain in force include upstream-tests-first, exact compiler-chain
identity, controlled three-track comparison, separate default and best results,
upstream-grade patches, complete bilingual documentation, and the code/test
necessity gate.
