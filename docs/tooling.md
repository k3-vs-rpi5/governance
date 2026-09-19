[中文](tooling.zh-CN.md)

# Script and tool register

The workspace accumulated scripts faster than it retired them, so the roles are
written down here once. Every entry states what it is, who invokes it, what it
overlaps with, and what should happen to it.

## The four classes

| class | meaning | consequence |
|---|---|---|
| **tool** | maintained, reached through a published door or a documented entry, with a defined input and output | gets fixed when it breaks; a new feature belongs here |
| **debug-essential** | not part of normal runs, but required to diagnose a class of failure we have actually hit | kept and documented; manual invocation expected; its cost is published |
| **frozen artefact** | produced the sealed evidence of one round and is kept only so that report can be regenerated | no features, no fixes beyond a syntax error; superseded tools are named |
| **temporary** | scratch that lives outside Git; may be deleted when its round closes | never referenced by a contract; listed here so it is not mistaken for a tool |

## Register

| path | class | reached through | overlaps with | action |
|---|---|---|---|---|
| `foundation/src/labctl/` | tool | the `labctl` entry point | - | keep |
| `*/scripts/build.sh`, `*/scripts/run.sh` | tool | the repository contract: exactly two doors | the ssh/credential block is repeated in each `run.sh` on purpose, so a repository stays self-contained when copied | keep |
| `hardware/k3-monitor/src/k3mon_project/payload_verify.py` | tool | `run.sh verify [payload]` | the only payload pre-flight check; written because a missing shared-library symlink invalidated a run | keep |
| `hardware/k3-monitor/src/k3mon_project/evidence_index.py` | tool | `run.sh evidence [root]` | the only index of the local evidence trees and their seals | keep |
| `hardware/k3-monitor/src/k3mon_project/selfcheck.py` | tool | `run.sh selfcheck` | runs the mechanical workspace checks in one pass, so the audit that found four defects does not have to be repeated by hand; the count is deliberately not written down here, because it keeps growing every time a check pays for itself twice | keep |
| `hardware/k3-monitor/src/k3mon_project/literature.py` | tool | `run.sh literature search\|verify\|platforms` | the published and the practised side of decode optimisation in one place: arXiv abstracts by id, arXiv search pages new-to-old, and the engineering sources this workspace measures itself against; written because the survey that asks "how does everyone else do this" was being redone by hand every round, and because the network here is partial, so which sources are reachable is itself part of the record | keep |
| `hardware/k3-monitor/src/k3mon_project/suite.sh` | tool | `scripts/run.sh build\|check\|scenarios\|suite\|analyse` | orchestrates the per-repository scripts; does not reimplement them | keep |
| `hardware/k3-fan-control/tests/guard-test.sh` | tool | `tests/guard-test.sh` | the only test of the fan guard that does not need the board; it fakes the sysfs tree inside a private mount namespace, and the board is unreachable for long stretches | keep |
| `projects/linux-6.18/src/linux_6_18_project/` | tool | `scripts/build.sh`, `scripts/run.sh` | the only place a kernel configuration, a build or a package version is assembled; without it a kernel change is a hand-copied image with no identity | keep |
| `hardware/k3-monitor/experiments/board.sh` | debug-essential | manual, and every experiment harness | the only ad-hoc board path; duplicates the credential block by design | keep |
| `hardware/k3-monitor/experiments/onnx-ep-analysis.py` | tool | `run.sh analyse` | supersedes `ai-optim.py`'s verdict checks | keep |
| `hardware/k3-monitor/experiments/onnx-ep-profile.py` | tool | manual | - | keep |
| `hardware/k3-monitor/experiments/onnx-ep-matrix.sh` | tool | manual | parallel in spirit to the llama.cpp matrix subcommand, different stack | keep |
| `hardware/k3-monitor/experiments/package-summary.py` | tool | manual | - | keep |
| `hardware/k3-monitor/experiments/phase-summary.py` | frozen artefact | - | round 14-15 phase split | keep, no features |
| `hardware/k3-monitor/experiments/phase-compare.py` | frozen artefact | - | superseded by the repeat table in `onnx-ep-analysis.py` | keep, no features |
| `hardware/k3-monitor/experiments/ai-scenarios.py` | frozen artefact | - | superseded by the scenario table in the suite | keep, no features |
| `hardware/k3-monitor/experiments/ai-optim.py` | frozen artefact | - | superseded by `onnx-ep-analysis.py` | keep, no features |
| `projects/llama.cpp/src/llama.cpp_project/bench.sh` | tool | `scripts/run.sh <subcommand>` | absorbs the three deleted experiment scripts | keep |
| `.../gen_kernel_probe.py` | tool | build and debug | - | keep |
| `.../probe/spacemit_kernel_probe.c` (+ template) | tool | `run.sh probe` | - | keep, cost published |
| `.../probe/libm_caller_probe.c` + `libm_versions.map` | debug-essential | manual | - | keep |
| `.../probe/fast_rounding.c` | frozen artefact | - | rejected optimisation, worked example for the triage rule | keep, never presented as a tool |
| `.../probe/fast_rounding_check.c` | debug-essential | manual | - | keep |
| `projects/spine-runtime/src/spine_runtime_project/a100_probe.cc` | tool | `scripts/run.sh` | - | keep |
| `.local/llvm.sh`, `.local/llama.cpp/{diag,test-dl-off,diagnostic-probe.patch}` | temporary | - | build-investigation scratch | delete when the round closes, with approval |
| `.local/coremark-work/` | frozen artefact | - | the campaign record that rounds cite | keep outside Git |

## Register granularity

A row may name a package directory (`foundation/src/labctl/`,
`projects/coremark/src/coremark_project/`) or a single file
(`experiments/onnx-ep-profile.py`). A package row covers every module and test
inside it: they are one tool with one role, and listing each module would hide
that. Code that is still unaudited is listed as such rather than omitted.

## Declared gaps

| gap | state |
|---|---|
| `projects/onnxruntime` has no `scripts/` doors and no `_project` package | declared: the provider is closed and its automation has not been written; the register says so instead of pretending the project is complete |
| neither `projects/onnxruntime` nor `projects/spine-runtime` has a project-local `AGENTS.md` | open: coremark and llama.cpp do |
| the six hardware repositories and three new projects are absent from `.repo/manifests/default.xml` | open: their remotes do not exist yet, so a manifest entry would break `repo sync`; entries are prepared as review material in each README |

## Rules that follow

- A repository exposes two doors; everything else it ships is reached through
  them. When a workflow needs a third verb, it becomes a subcommand, not a
  script.
- Ad-hoc device access is one tool (`board.sh`); the duplication inside each
  `run.sh` is deliberate self-containment, not a merge candidate.
- A tool that produced sealed evidence is frozen rather than deleted, and its
  successor is named in place. Deleting it would break the ability to regenerate
  that evidence.
- Temporary items live outside Git, are named in this register, and are removed
  at the end of their round - with human approval, because cleanup is a
  destructive action under the workspace contract.
- Overlap is resolved by naming the survivor: `onnx-ep-analysis.py` for verdict
  checking, `package-summary.py` for a single package, `bench.sh` for llama.cpp
  measurement.
