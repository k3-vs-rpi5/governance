[中文](design.zh-CN.md)

# Validation-system design

## Objective

The lab compares Raspberry Pi 5 with SpacemiT K3 Pico under controlled,
auditable conditions and turns verified K3 improvements into upstream-ready
patches. It reports performance only; externally measured power evidence may be
attached without changing the software result contract.

## Bottom-up dependency order

Optimization follows the software dependency graph. The foundation is accepted
before a project is selected; low-level runtime and library work precedes
dependent applications. A component enters the roadmap only after its official
or shipped correctness tests, measurement method, environmental controls,
failure policy, and raw-evidence format are defined.

The sequence is fixed:

1. qualify installed OS identities, runtime profiles, compiler chains, affinity, and telemetry;
2. qualify CPU, memory, storage-exclusion, and network controls;
3. validate a leaf library or official suite with its own tests;
4. locate hotspots and state a falsifiable optimization mechanism;
5. screen candidates, promote statistically credible results, and land patches;
6. validate dependent libraries and applications only after the lower layer is stable.

## Three-track control model

Each project separates `rpi-native`, `k3-compatible`, and `k3-development`.
The compatible K3 track runs a locked low-version Debian rootfs and compiler
chain matching Raspberry Pi as closely as published architecture packages
allow. The development K3 track uses the current Bianbu runtime and newer
compiler. CPU sets are explicitly bound to one or four cores; storage-intensive
phases are excluded from timed regions because the current K3 uses UFS 2.2 and
Raspberry Pi uses SD storage.

## One project cycle

One cycle contains preregistration, default-parity measurement, hotspot
evidence, candidate validation, formal confirmation, patch landing, independent
reproduction, and bilingual reporting. `default-parity` keeps source, port,
flags class, parameters, seeds, duration, CPU set, and telemetry equal.
`best-achievable` is a separately labelled K3 optimization result. The CoreMark
K3 research target remains `7.7+ CoreMark/MHz`; reaching normal performance is
not completion by itself.

## Trust boundary

Official or upstream-shipped suites are the default. A local test is accepted
only when the upstream test is inadequate and the local test has adversarial
fixtures, negative paths, deterministic results, peer review, and a documented
oracle. Scripts never invent a replacement benchmark when an official suite
exists. Credentials, downloaded toolchains, raw runs, and recovery artifacts
stay outside Git.

## Publication model

Every claim resolves to an immutable raw run, exact source and patch identities,
compiler/linker/libc versions, runtime-profile digest, environmental samples,
and the public build/run entry points. Governance, foundation, and every project
have independent Git histories selected by a Google `repo` release manifest.
Reports live in a separate non-default repository. Each project has one stable
logical report updated in place; dated local evidence remains append-only. A
clean default synchronization contains no reports and must reproduce the
accepted code version set before the first public push.
