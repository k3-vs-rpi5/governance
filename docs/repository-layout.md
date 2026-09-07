[中文](repository-layout.zh-CN.md)

# Repository layout standard

## Repo workspace root

The `k3Vrp5` root is a Google `repo` workspace, not a Git repository. Only
`.repo/` and manifest-generated root files live directly at this level.

```text
k3Vrp5/
├── .repo/
├── AGENTS.md
├── AGENTS.zh-CN.md
├── governance/
├── foundation/
├── projects/
│   └── coremark/
└── reports/                    optional; absent from default synchronization
```

The manifest copies the root collaboration pair from governance. No credential,
cache, build directory, raw candidate, or report is owned by the workspace root.

## Manifest repository

```text
manifest/
├── README.md
├── README.zh-CN.md
├── default.xml
└── releases/
    └── coremark-baseline.xml
```

`default.xml` may follow reviewed development branches. Files below `releases/`
pin exact project commit IDs. Reports use groups `reports,notdefault`.

## Governance repository

```text
governance/
├── AGENTS.md
├── AGENTS.zh-CN.md
├── README.md
├── README.zh-CN.md
├── config/
│   ├── documentation.yaml
│   └── schemas/
│       └── documentation.schema.yaml
└── docs/
    ├── design.md
    ├── design.zh-CN.md
    ├── operations.md
    ├── operations.zh-CN.md
    └── <other-canonical-document-pairs>
```

Governance contains policy and review records only. It does not contain device
runtime code, benchmark patches, reports, or raw evidence.

## Foundation repository

```text
foundation/
├── README.md
├── README.zh-CN.md
├── pyproject.toml
├── config/
│   ├── environments/
│   ├── runtime-profiles/
│   └── schemas/
├── environments/
│   └── base/
├── src/
│   └── labctl/
└── tests/
```

Foundation owns only shared environment, device, telemetry, statistics,
evidence, documentation, and report-contract behavior. A benchmark-specific
parser or optimization belongs to its project repository.

## Project repository

Each project is one Git repository. CoreMark is not nested beneath a generic CPU
project.

```text
coremark/
├── README.md
├── README.zh-CN.md
├── project.yaml
├── sources.lock.yaml
├── environments.lock.yaml
├── scripts/
│   ├── build.sh
│   └── run.sh
├── patches/
│   ├── README.md
│   ├── README.zh-CN.md
│   ├── manifest.yaml
│   └── 0001-*.patch
├── src/
│   └── coremark_project/
└── tests/
    ├── test_delivery_contract.py
    └── test_patch_contract.py
```

The `scripts/` directory contains exactly two Shell files. Reproduction usage
belongs in the project README pair. Redundant READMEs, candidate-matrix scripts,
historical diagnostics, and a test file per implementation module are forbidden
unless they independently pass the code-necessity gate.

## Reports repository

```text
reports/
├── README.md
├── README.zh-CN.md
└── coremark/
    ├── report.md
    ├── report.zh-CN.md
    ├── report.yaml
    └── evidence/
        ├── index.yaml
        └── <content-digest>/
```

There is one logical report per project. Do not create cycle, date, `latest`,
`related`, or revision directories. Git history provides inheritance. Compact
accepted evidence may be tracked by digest; full large logs remain in an
external archive referenced by stable locator, SHA-256, and byte size.

## Local-only data

Credentials, private inventory, recovery bundles, downloaded suites and
toolchains, build directories, device synchronization areas, and candidate raw
evidence are outside all managed Git repositories. Their location is an
operator-selected absolute path passed to the public project scripts. A formal
run fails closed if that root is missing, relative, inside a project Git tree,
or reuses an existing run directory.

## Naming and update rules

- Git repository and project IDs use stable lowercase kebab-case.
- English project-owned Markdown is `name.md`; Chinese is `name.zh-CN.md`.
- The `.en.md` suffix is forbidden.
- Report filenames are stable and updated in place.
- Raw run IDs use UTC and are append-only; they are not report filenames.
- Patches use ordered four-digit prefixes and an upstream-style subject.
- Toolchains, source revisions, runtime profiles, and evidence are content
  locked with exact version plus digest.

## Enforcement

Use existing official tools where possible: XML parsing and `repo manifest` for
workspace composition, Git for history and ignored-file boundaries, upstream
tests for benchmark correctness, ShellCheck for public Shell entries, and the
smallest foundation validators for cross-repository evidence contracts. Do not
create a parallel layout framework merely to check this document.
