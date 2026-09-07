[中文](README.zh-CN.md)

# K3 and Raspberry Pi 5 Reproducible Performance Lab

## Purpose

This Google `repo` workspace builds a public, reproducible validation system for comparing
the SpacemiT K3 Pico with Raspberry Pi 5 and for optimizing K3 software. The
first independent project is CoreMark. Governance, shared foundation, projects,
and reports have separate Git histories. Performance is collected by software;
power is measured externally under the same documented platform controls.

## Fixed comparison model

Every report keeps three tracks separate: `rpi-native`, `k3-compatible`, and
`k3-development`. Each optimization project publishes both `default-parity`
and `best-achievable`; optimized K3 results never replace the equal-parameter
baseline. CPU affinity, frequency, temperature, memory, runtime, exact compiler
chain, storage, wired network, commands, and validation state are recorded.

## Repository entry points

- [Workspace architecture](docs/repo-workspace-architecture.md) defines the
  Google `repo` topology and migration boundary.
- [Repository contract](docs/repository-contract.md) defines mandatory policy.
- [Project requirements](docs/project-requirements.md) defines the test cycle.
- [Repository layout](docs/repository-layout.md) defines canonical paths.
- [Operations](docs/operations.md) defines local and device execution.
- The platform baseline in the synchronized foundation repository separates
  vendor specifications, board inventory, and laboratory controls.

## Local verification

Create an isolated foundation development environment and run only its retained
contract tests:

```bash
python3 -m venv foundation/.venv
foundation/.venv/bin/python -m pip install -e 'foundation[dev]'
foundation/.venv/bin/python -m pytest foundation/tests
foundation/.venv/bin/python -m ruff check foundation/src foundation/tests
projects/coremark/scripts/build.sh --help
projects/coremark/scripts/run.sh --help
```

Public project execution uses only the project-owned `scripts/build.sh` and
`scripts/run.sh`. Missing build dependencies and compiler packages are installed
through APT at declared versions; downloaded sources, compilers, credentials,
raw runs, and power images remain local unless the contract explicitly permits
the artifact.

## Release state

No public result or first push is permitted until the component checks and
device gates pass, an immutable release manifest recreates a clean default
workspace without reports, any report promotion is independently reviewed, and
the exact remote URL, branch, and refspec are approved. No Git remote is
configured by this workspace contract.
