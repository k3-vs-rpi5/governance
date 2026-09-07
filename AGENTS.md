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
- Prepare accepted source changes as reproducible, zero-fuzz, upstream-grade
  patches.

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
