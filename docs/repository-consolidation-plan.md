[中文](repository-consolidation-plan.zh-CN.md)

# Repo Workspace Migration Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use
> `superpowers:subagent-driven-development` or `superpowers:executing-plans` to
> implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for
> tracking.

**Goal:** Convert the current monorepo into a verified Google `repo` workspace
whose governance, foundation, CoreMark, and reports components have independent
Git histories, while reports remain excluded from default migration.

**Architecture:** The current worktree is a recovery source, not the final
repository boundary. Content is normalized in place, partitioned into candidate
component trees, verified, committed once into local independent repositories,
and synchronized into a clean workspace through a sibling-relative manifest.
The reports project is pinned but belongs to `reports,notdefault`.

**Tech Stack:** Google `repo`, Git, Python 3.11+, PyYAML 6.x, pytest 8.x,
Ruff, POSIX Shell, ShellCheck, APT, CoreMark 1.0, Debian 13 riscv64 rootfs,
Bianbu 4.0.4, and pinned GCC/Clang toolchains.

**Spec:** [repo-workspace-architecture.md](repo-workspace-architecture.md)

## Global constraints

- Do not push any repository.
- Do not rewrite or discard the current worktree, index, ignored evidence, or
  mode-`0600` `.device-credentials` before a verified recovery capture exists.
- The first commit in each target repository is created only after its source
  tree passes the applicable basic checks.
- The default synchronized workspace must not contain `reports/`.
- No final manifest contains a machine-specific absolute URL or placeholder.
  The common remote uses a sibling-relative fetch URL; local absolute URLs are
  confined to ignored migration evidence.
- CoreMark is project ID `coremark`; `cpu-foundation` is not a target project.
- Every project has one `build.sh`, one `run.sh`, one stable logical report, and
  no date-versioned report copies.
- Raspberry Pi 5 is the script-validation origin. K3 Bianbu is the primary
  development environment. Formal work preserves `rpi-native`,
  `k3-compatible`, and `k3-development` plus separate `default-parity` and
  `best-achievable` results.
- Use official or upstream correctness suites first. Local tests protect only
  enduring repository-owned contracts; no test-count or coverage target exists.
- Missing packages are installed through APT. Compiler, linker, libc, runtime,
  source, flags, and patch identities are recorded explicitly.
- Use paired complete English and Chinese Markdown. Do not create additional
  prose files when an existing canonical pair can own the content.
- Use `apply_patch` for deliberate content edits. Mechanical tree partitioning
  may use `rsync` after source and destination lists are reviewed.

---

### Task 1: Freeze and inventory the approved source worktree

**Files:**

- Create locally: `.local/repo-migration/recovery/pre-migration.bundle`
- Create locally: `.local/repo-migration/recovery/status.txt`
- Create locally: `.local/repo-migration/recovery/index.diff`
- Create locally: `.local/repo-migration/recovery/worktree.diff`
- Create locally: `.local/repo-migration/inventory/path-owners.tsv`

**Interfaces:**

- Consumes: branch `image-delivery-baseline`, its index, unstaged operations
  updates, ignored evidence, and the approved architecture pair.
- Produces: a verified recovery point and a complete path-to-target-owner map.

- [x] **Step 1: Capture identity without reading secret contents**

  Record the output of these commands beneath
  `.local/repo-migration/recovery/`:

  ```bash
  git rev-parse HEAD
  git status --short
  git diff --cached --binary
  git diff --binary
  git ls-files
  ```

  Record only the path, mode, and ignore status of `.device-credentials`, never
  its values.

- [x] **Step 2: Create and verify the recovery bundle**

  Run these commands and save verification output locally:

  ```bash
  git bundle create .local/repo-migration/recovery/pre-migration.bundle --all
  git bundle verify .local/repo-migration/recovery/pre-migration.bundle
  ```

- [x] **Step 3: Record the executable baseline**

  Run `python3 -m pytest -q`, `python3 -m ruff check src tests projects`,
  `sh -n` for the two CoreMark entry points, ShellCheck, and the zero-fuzz
  CoreMark patch round trip. Save exact output and exit status locally.

- [x] **Step 4: Classify every retained tracked path**

  Write `path-owners.tsv` with exactly one owner from `manifest`, `governance`,
  `foundation`, `coremark`, `reports`, or `drop`. Mark current CoreMark code and
  patches `coremark`, generic runtime/environment logic `foundation`, policy
  `governance`, report/evidence material `reports`, and obsolete one-time
  consolidation machinery `drop`.

- [x] **Step 5: Checkpoint**

  Verify all recovery files are ignored, the bundle is readable, and no Git
  remote or tracked file changed during capture.

### Task 2: Make governance the single collaboration and policy authority

**Files:**

- Create: `AGENTS.md`
- Create: `AGENTS.zh-CN.md`
- Modify: `README.md`
- Modify: `README.zh-CN.md`
- Modify: `docs/design.md`
- Modify: `docs/design.zh-CN.md`
- Modify: `docs/operations.md`
- Modify: `docs/operations.zh-CN.md`
- Modify: `docs/project-lifecycle.md`
- Modify: `docs/project-lifecycle.zh-CN.md`
- Modify: `docs/project-requirements.md`
- Modify: `docs/project-requirements.zh-CN.md`
- Modify: `docs/repository-contract.md`
- Modify: `docs/repository-contract.zh-CN.md`
- Modify: `docs/repository-layout.md`
- Modify: `docs/repository-layout.zh-CN.md`
- Modify: `config/documentation.yaml`

**Interfaces:**

- Consumes: the approved architecture and existing durable validation rules.
- Produces: migration-safe policy that contains no `cpu-foundation`, monorepo
  report path, two-commit-history, or chat-only authority assumption.

- [x] **Step 1: Write the root collaboration contract**

  Add a concise `AGENTS.md` and complete Chinese counterpart. Require reading
  governance/project contracts, the code-necessity question, control-variable
  validation, upstream-tests-first, evidence before claims, human approval for
  publication/destructive actions/remotes/push, and no secret disclosure.

- [x] **Step 2: Rewrite existing canonical documents instead of adding guides**

  Merge the approved repo topology, reports separation, one-project-one-report,
  three-track environment, child-project cycle, and minimal-test rules into the
  existing canonical pairs. Remove the old `cpu-foundation` and in-monorepo
  reports model.

- [x] **Step 3: Update the documentation catalog**

  Register the architecture and `AGENTS` pairs, increment revisions for changed
  pairs, and set Chinese review status consistently. Do not add a documentation
  test for the policy prose itself.

- [x] **Step 4: Verify governance**

  Run `git diff --check`, the existing documentation checker, a local-link
  check, and an exact search for obsolete normative strings. Expected obsolete
  references may remain only in the architecture migration-consequences section
  or ignored recovery evidence.

### Task 3: Contract the foundation implementation and test surface

**Files:**

- Delete: `src/labctl/coremark.py`
- Delete: `src/labctl/conformance.py`
- Delete: `src/labctl/readiness.py`
- Delete: `src/labctl/release.py`
- Delete: `tests/test_coremark_summary.py`
- Delete: `tests/test_conformance.py`
- Delete: `tests/test_readiness.py`
- Delete: `tests/test_release.py`
- Modify: `src/labctl/cli.py`
- Modify: `src/labctl/documentation.py`
- Modify: `src/labctl/governance.py`
- Modify: `src/labctl/layout.py`
- Modify: `tests/test_cli.py`
- Modify: `tests/test_documentation.py`
- Modify: `tests/test_governance.py`
- Modify: `tests/test_layout.py`
- Modify: `pyproject.toml`

**Interfaces:**

- Consumes: the new separate-repository contract.
- Produces: `labctl` as a shared environment/evidence validator without
  CoreMark-specific parsing or one-time monorepo-history machinery.

- [x] **Step 1: Record a necessity disposition for every current test file**

  In the implementation review record, map each retained file to one enduring
  contract and deletion condition. Remove files serving only CoreMark,
  `cpu-foundation`, in-tree reports, one-time consolidation, or the old history
  shape. Do not create a checker that tests this review record.

- [x] **Step 2: Make the smallest existing tests fail for the new boundaries**

  Change the layout fixture to accept a standalone project root with exactly
  `build.sh` and `run.sh`; change report validation to accept the reports
  repository root with `coremark/report.md`, `report.zh-CN.md`, `report.yaml`,
  and `evidence/index.yaml`. Remove assertions for a separate reproduction
  report and monorepo-relative project paths.

- [x] **Step 3: Run the focused tests and observe the old assumptions fail**

  Run `pytest tests/test_layout.py tests/test_governance.py tests/test_cli.py -q`.
  Expected failures must mention old report/project path or removed command
  behavior, not unrelated regressions.

- [x] **Step 4: Remove obsolete modules and simplify shared validators**

  Delete the four obsolete modules and their dedicated tests. Remove
  `coremark-summary`, clean-history release scopes, and hard-coded
  `cpu-foundation` readiness from the CLI. Keep environment, runtime,
  compatibility, telemetry, statistics, remote transport, suite-lock, and
  append-only evidence behavior only where an existing test protects a declared
  shared contract.

- [x] **Step 5: Collapse duplicate contract cases**

  Parameterize equivalent documentation/layout/report failures and remove tests
  for wording, static values, wrapper internals, or behavior already checked by
  schemas or a lower layer. Do not replace deleted tests with a new framework.

- [x] **Step 6: Verify foundation**

  Run the retained foundation tests, Ruff, and package build. Confirm no source
  or test imports `labctl.coremark`, `labctl.conformance`, `labctl.readiness`, or
  `labctl.release`.

### Task 4: Extract CoreMark as a path-independent project

**Files:**

- Rename: `projects/cpu-foundation/` to `projects/coremark/`
- Rename: `projects/coremark/src/cpu_foundation/` to
  `projects/coremark/src/coremark_project/`
- Create: `projects/coremark/sources.lock.yaml`
- Create: `projects/coremark/environments.lock.yaml`
- Modify: `projects/coremark/project.yaml`
- Modify: `projects/coremark/README.md`
- Modify: `projects/coremark/README.zh-CN.md`
- Modify: `projects/coremark/scripts/build.sh`
- Modify: `projects/coremark/scripts/run.sh`
- Modify: `projects/coremark/src/coremark_project/cli.py`
- Modify: `projects/coremark/src/coremark_project/delivery.py`
- Modify: `projects/coremark/src/coremark_project/manifests.py`
- Consolidate: `projects/coremark/tests/test_delivery_contract.py`
- Retain and simplify: `projects/coremark/tests/test_patch_contract.py`
- Delete after merging unique content: `benchmarks/coremark-candidates.yaml`,
  `benchmarks/free-baseline*`, `benchmarks/suite-manifest.yaml`, redundant
  directory READMEs, and separate reproduction documents.

**Interfaces:**

- Produces: project ID `coremark`, package `coremark_project`, explicit
  `COREMARK_EVIDENCE_ROOT`, and exactly two public Shell entry points.
- Consumes: foundation through the manifest workspace and locked environment
  profile IDs, never through a hard-coded monorepo root.

- [x] **Step 1: Write the two minimal project contract tests first**

  `test_delivery_contract.py` verifies only the two executable entry points,
  project-root discovery, explicit evidence-root containment, official CRC and
  minimum-duration rejection, and compatible-rootfs isolation.
  `test_patch_contract.py` verifies manifest identity plus one real zero-fuzz
  apply/reverse round trip. Remove the test that opens an in-tree report.

- [x] **Step 2: Run the focused tests and observe failure**

  Run `PYTHONPATH=projects/coremark/src:src pytest projects/coremark/tests -q`.
  Expected failures identify old package names, old absolute-root derivation,
  or old evidence paths.

- [x] **Step 3: Rename and minimize the project content**

  Replace every normative `cpu-foundation` identity with `coremark`. Merge
  unique benchmark, optimization, script, and test README content into the
  project README or patch README; remove the now-redundant files.

- [x] **Step 4: Make path and environment contracts explicit**

  Derive `PROJECT_ROOT` from the installed project package. Require an absolute
  `COREMARK_EVIDENCE_ROOT` outside tracked project content. Resolve foundation
  profiles by locked ID/digest from the synchronized workspace. Fail closed on
  a missing profile, digest mismatch, output reuse, host-library leakage, or
  undeclared compiler fallback.

- [x] **Step 5: Correct execution order and metadata**

  Run upstream `make check` and official correctness mode before performance;
  apply declared warmups; record run ID, boot ID, compiler/linker/libc/binutils,
  full flags, affinity, sampled frequency, average temperature, memory/swap,
  storage, network, throttling, services, and failure state. Keep quick results
  non-reportable and formal results append-only.

- [x] **Step 6: Verify the public delivery**

  Run project pytest, Ruff, `sh -n`, ShellCheck, upstream CoreMark `make check`,
  patch apply/reverse, and both `--help` commands. Confirm exactly two `*.sh`
  files exist and neither script writes a report.

### Task 5: Reshape reports into one project/one report

**Files:**

- Move: `reports/cpu-foundation/coremark-three-track/report.md` to
  `reports/coremark/report.md`
- Move: `reports/cpu-foundation/coremark-three-track/report.zh-CN.md` to
  `reports/coremark/report.zh-CN.md`
- Move and rewrite: `reports/cpu-foundation/coremark-three-track/report.yaml` to
  `reports/coremark/report.yaml`
- Create: `reports/coremark/evidence/index.yaml`
- Delete after merging: CoreMark `reproduction.md` pair and non-project
  governance/infrastructure reports
- Modify: `reports/README.md`
- Modify: `reports/README.zh-CN.md`

**Interfaces:**

- Produces: one stable CoreMark report pair and a content-addressed public
  evidence index independent of code migration.

- [x] **Step 1: Inventory every claim and referenced run before moving files**

  Mark each claim `verified-local`, `missing-evidence`, or `blocked-retest`.
  Never replace a missing run with a similarly named run. Preserve unavailable
  conclusions as limitations, not current public measurements.

- [x] **Step 2: Merge reproduction content into the single report and project README**

  Keep public result interpretation and evidence links in the report; keep
  build/run instructions in the CoreMark project README. Delete the separate
  reproduction pair after both destinations contain its unique information.

- [x] **Step 3: Build the machine-readable report identity**

  Record exact manifest, governance, foundation, project, source, toolchain,
  patch, and evidence identities. Store accepted compact evidence by content
  digest. For external full logs, record locator, SHA-256, bytes, and
  availability.

- [x] **Step 4: Validate the report independently**

  Run the foundation report/evidence validator against a reports-only fixture
  and the real `reports/coremark` tree. Confirm the report cannot import project
  source or depend on the current monorepo path.

### Task 6: Define the manifest and component partition

**Files:**

- Create: `manifests/README.md`
- Create: `manifests/README.zh-CN.md`
- Create: `manifests/default.xml`
- Generate after component commits: `manifests/releases/coremark-baseline.xml`
- Create locally: `.local/repo-migration/component-trees/`

**Interfaces:**

- Produces: a sibling-relative Google `repo` manifest with projects
  `governance.git`, `foundation.git`, `coremark.git`, and `reports.git`.

- [x] **Step 1: Assemble reviewed component trees mechanically**

  Use `path-owners.tsv` and `rsync` to create governance, foundation, coremark,
  and reports trees below the ignored migration directory. Compare each
  destination file list against the inventory; reject duplicate or ownerless
  paths.

- [x] **Step 2: Write the development manifest**

  Use one remote whose fetch path is relative to the manifest repository URL.
  Check out governance at `governance`, foundation at `foundation`, CoreMark at
  `projects/coremark`, and reports at `reports`. Assign reports to
  `reports,notdefault`. Copy governance `AGENTS.md` and `AGENTS.zh-CN.md` to the
  workspace root.

- [x] **Step 3: Validate XML and policy without new test code**

  Parse the XML with Python's standard library, inspect project paths/groups,
  and confirm there is no absolute local URL, placeholder, duplicate path, or
  reports project in the default group.

### Task 7: Validate, create clean component histories, and rehearse migration

**Files:**

- Create locally: `.local/repo-migration/sources/<component>/`
- Create locally: `.local/repo-migration/remotes/<component>.git`
- Create locally: `.local/repo-migration/workspace/`
- Create locally: `.local/repo-migration/evidence/`

**Interfaces:**

- Consumes: the four component trees and manifest tree.
- Produces: five local independent Git histories and a clean synchronized
  workspace with reports absent by default.

- [x] **Step 1: Run basic checks before the first commits**

  Governance passes documentation/link checks; foundation passes retained tests,
  Ruff, and package build; CoreMark passes project tests, ShellCheck, upstream
  correctness, and patch replay; reports passes bilingual and evidence checks;
  manifest passes XML/policy inspection.

- [x] **Step 2: Create one clean root commit per candidate repository**

  Initialize `main`, add only reviewed component content, and inspect:

  ```bash
  git status --short
  git diff --cached --check
  ```

  Then commit once with component-specific messages. Do not configure or push a
  network remote.

- [x] **Step 3: Create local bare validation remotes**

  Clone each repository as a bare sibling below
  `.local/repo-migration/remotes/`. Initialize the manifest repository last so
  its sibling-relative remote resolves all components.

- [x] **Step 4: Synchronize a fresh default workspace**

  Run `repo init` from the local manifest bare repository and `repo sync` in the
  ignored workspace. Verify governance, foundation, and
  `projects/coremark` commits exactly match the manifest and `reports/` is
  absent.

- [x] **Step 5: Verify collaboration discovery and explicit reports sync**

  Confirm root `AGENTS.md` matches governance. Explicitly select the reports
  group, verify the pinned reports commit and CoreMark report, then remove that
  checkout and prove a new default sync again omits reports.

- [x] **Step 6: Generate the immutable release manifest**

  Export a revision-pinned manifest, save it as
  `manifests/releases/coremark-baseline.xml`, validate it in a second clean
  workspace, and update the manifest repository's first commit only if the
  release file was not already included. Do not create a second public-history
  commit merely to record migration diagnostics.

### Task 8: Device qualification and completion gate

**Files:**

- Create locally: `<evidence-root>/foundation/<run-id>/`
- Create locally: `<evidence-root>/coremark/<run-id>/`
- Update only after accepted formal evidence: candidate reports repository
  `coremark/report.md`, `coremark/report.zh-CN.md`, `coremark/report.yaml`, and
  `coremark/evidence/index.yaml`

**Interfaces:**

- Produces: qualified three-track evidence or an explicit blocked state without
  a false public claim.

- [ ] **Step 1: Qualify both devices without exposing credentials**

  Load the ignored credential file, connect over SSH, record board identity,
  boot ID, OS/kernel, wired network, CPU topology, one-core/four-core sets,
  frequency/governor, temperature, memory/swap, storage, services, throttling,
  and toolchain/runtime identities. Do not print password values.

- [ ] **Step 2: Verify the environment matrix**

  Validate `rpi-native`, the K3 locked compatible rootfs/chroot, and
  `k3-development`. Produce an explicit deviation list. Any material unresolved
  divergence prevents the label strictly aligned.

- [ ] **Step 3: Run CoreMark quick qualification**

  On Raspberry Pi 5 validate both public scripts first. Run default-parity on
  Raspberry Pi 5 and both K3 tracks, then K3 best-achievable. Require official
  CRCs, at least ten seconds, declared warmups, stable telemetry, and sealed
  append-only evidence.

- [ ] **Step 4: Decide formal readiness**

  If a device resets, becomes unreachable, changes identity, drifts in runtime,
  or fails stability gates, write `blocked` evidence and stop formal promotion.
  Otherwise run the preregistered formal sessions and validate every package.

- [ ] **Step 5: Final verification without push**

  Re-run all component checks and the pinned release-manifest synchronization.
  Confirm zero configured network remotes in the candidate source repositories,
  no reports in the default workspace, no secret literals in tracked content,
  and no unsupported public claim. Report exact remaining blockers, especially
  missing final remote URLs or unavailable hardware.

### Task 9: Publish the canonical GitHub workspace

**Files:**

- Modify: `README.md`
- Modify: `README.zh-CN.md`
- Modify: `docs/repo-workspace-architecture.md`
- Modify: `docs/repo-workspace-architecture.zh-CN.md`
- Modify: `config/documentation.yaml`
- Modify in manifests: `README.md`
- Modify in manifests: `README.zh-CN.md`
- Remove from manifests: `tools/provision_gitea.py`
- Remove from manifests: `tests/test_provision_gitea.py`

**Interfaces:**

- Consumes: the five validated component histories and the approved GitHub
  organization `k3-vs-rpi5`.
- Produces: five public sibling repositories, a canonical public `repo init`
  URL, retained private Gitea mirror remotes, and fresh-clone evidence.

- [x] **Step 1: Re-run the public-history safety gate**

  Confirm the GitHub organization membership is active with administrator
  authority, all five target names are unused, every component worktree is
  clean, and every reachable commit is free of device credentials, secret
  values, local evidence, caches, builds, and recovery material. Run each
  component's existing focused checks; do not add migration-only tests.

- [x] **Step 2: Update the canonical bilingual documentation**

  Declare `https://github.com/k3-vs-rpi5` as the canonical public host. Keep the
  repositories named `governance`, `foundation`, `coremark`, `reports`, and
  `manifests`, with reports excluded from default synchronization. Replace the
  manifest placeholder with:

  ```bash
  repo init -u https://github.com/k3-vs-rpi5/manifests.git -b main -m default.xml
  ```

  Remove the Gitea provisioning script and its migration-only test. Retain the
  existing Gitea repositories only as operator-configured private mirrors.

- [ ] **Step 3: Create and publish the five GitHub repositories**

  Create empty public repositories under `k3-vs-rpi5` without generated files.
  In each local component repository rename the approved Gitea `origin` to
  `gitea`, add `https://github.com/k3-vs-rpi5/<name>.git` as `origin`, and push
  only `refs/heads/main:refs/heads/main`. Verify the remote object identity and
  default branch after every push.

- [ ] **Step 4: Apply the initial branch safety policy**

  Protect `main` against force pushes and deletion while retaining ordinary
  direct pushes. Do not require reviews or status checks until those controls
  have real maintainers and workflows; a nominal rule that blocks the sole
  maintainer is not an integrity improvement.

- [ ] **Step 5: Reconstruct from the public host**

  In fresh temporary directories, run the canonical default manifest and prove
  that governance, foundation, and CoreMark synchronize while reports remains
  absent. Then synchronize `default,reports` and prove the reports repository
  appears. Finally initialize the pinned release manifest and verify every
  checkout matches its declared full commit.

- [ ] **Step 6: Audit the published state**

  Verify that all five repositories are public, use `main`, contain the expected
  commit, have force-push/deletion protection, and expose no unexpected branch
  or tag. Re-run the secret scan against reachable GitHub objects and record the
  exact canonical URLs in the handoff.

## Completion definition

The implementation is complete when a clean default `repo` workspace can be
reconstructed from the public GitHub manifest, the reports repository is
separately auditable, CoreMark is an independent minimally tested project, all
three environment tracks are either qualified or explicitly blocked, and the
original monorepo and private Gitea mirrors remain recoverable.
