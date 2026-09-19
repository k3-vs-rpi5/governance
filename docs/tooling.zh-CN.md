[English](tooling.md)

# 脚本与工具登记表

工作区积累脚本的速度快于淘汰它们的速度，因此这里把角色一次写清。每一条都说明：它是
什么、由谁调用、与谁重叠、以及应当如何处理。

## 四个类别

| 类别 | 含义 | 后果 |
|---|---|---|
| **工具** | 受维护、通过公开入口或已记录的入口触达、有明确的输入与输出 | 坏了要修；新功能归到这里 |
| **调试必备** | 不属于日常运行，但用于诊断我们**实际遇到过**的那类故障 | 保留并记录；预期手动调用；其代价要公布 |
| **冻结产物** | 为某一轮产出封存证据，仅为再生成那份报告而保留 | 不加功能、除语法错误外不修；并指明被谁取代 |
| **临时** | 位于 Git 之外的草稿；该轮结束后可删除 | 绝不被契约引用；登记在此以免被误当成工具 |

## 登记表

| 路径 | 类别 | 触达方式 | 与谁重叠 | 处理 |
|---|---|---|---|---|
| `foundation/src/labctl/` | 工具 | `labctl` 入口 | - | 保留 |
| `*/scripts/build.sh`、`*/scripts/run.sh` | 工具 | 仓库契约：恰好两个入口 | 每个 `run.sh` 里重复的 ssh/凭据块是**故意**的，以便仓库被复制后自洽 | 保留 |
| `hardware/k3-monitor/src/k3mon_project/payload_verify.py` | 工具 | `run.sh verify [payload]` | 唯一的载荷交付前校验；因一次缺符号链接导致运行作废而写 | 保留 |
| `hardware/k3-monitor/src/k3mon_project/evidence_index.py` | 工具 | `run.sh evidence [root]` | 唯一的本地证据树与封存状态索引 | 保留 |
| `hardware/k3-monitor/src/k3mon_project/selfcheck.py` | 工具 | `run.sh selfcheck` | 一次跑完机械自检，使那套找出四个缺陷的审计不必手工重复；这里特意不写检查项数量，因为每有一项检查第二次证明自己的价值，它就会增加 | 保留 |
| `hardware/k3-monitor/src/k3mon_project/literature.py` | 工具 | `run.sh literature search\|verify\|platforms` | 把解码优化的"已发表"与"已实践"两侧放在一处：按 id 取 arXiv 摘要、按新旧顺序查 arXiv 搜索页，以及本工作区用来对照的工程来源；写它的原因是"别人怎么做"这类调研每轮都被手工重做，且本机网络是部分可达的，哪些来源可读本身也是记录的一部分 | 保留 |
| `hardware/k3-monitor/src/k3mon_project/suite.sh` | 工具 | `scripts/run.sh build\|check\|scenarios\|suite\|analyse` | 编排各仓库脚本，不重复实现 | 保留 |
| `hardware/k3-fan-control/tests/guard-test.sh` | 工具 | `tests/guard-test.sh` | 唯一不需要板卡的风扇守护进程测试；它在私有 mount namespace 中伪造 sysfs 树，而板卡会长时间不可达 | 保留 |
| `projects/linux-6.18/src/linux_6_18_project/` | 工具 | `scripts/build.sh`、`scripts/run.sh` | 唯一组装内核配置、构建与包版本的地方；没有它，内核变更就是一张没有身份的拷贝镜像 | 保留 |
| `hardware/k3-monitor/experiments/board.sh` | 调试必备 | 手动，以及各实验装置 | 唯一的临时板卡通道；按设计重复凭据块 | 保留 |
| `hardware/k3-monitor/experiments/onnx-ep-analysis.py` | 工具 | `run.sh analyse` | 取代 `ai-optim.py` 的结论复核 | 保留 |
| `hardware/k3-monitor/experiments/onnx-ep-profile.py` | 工具 | 手动 | - | 保留 |
| `hardware/k3-monitor/experiments/onnx-ep-matrix.sh` | 工具 | 手动 | 与 llama.cpp 的 matrix 子命令思路相同，但栈不同 | 保留 |
| `hardware/k3-monitor/experiments/package-summary.py` | 工具 | 手动 | - | 保留 |
| `hardware/k3-monitor/experiments/phase-summary.py` | 冻结产物 | - | 第 14-15 轮的阶段切分 | 保留，不加功能 |
| `hardware/k3-monitor/experiments/phase-compare.py` | 冻结产物 | - | 已被 `onnx-ep-analysis.py` 的重复表取代 | 保留，不加功能 |
| `hardware/k3-monitor/experiments/ai-scenarios.py` | 冻结产物 | - | 已被套件的场景表取代 | 保留，不加功能 |
| `hardware/k3-monitor/experiments/ai-optim.py` | 冻结产物 | - | 已被 `onnx-ep-analysis.py` 取代 | 保留，不加功能 |
| `projects/llama.cpp/src/llama.cpp_project/bench.sh` | 工具 | `scripts/run.sh <subcommand>`（子命令形式） | 吸收了被删掉的三个实验脚本 | 保留 |
| `.../gen_kernel_probe.py` | 工具 | 构建与调试 | - | 保留 |
| `.../probe/spacemit_kernel_probe.c`（含模板） | 工具 | `run.sh probe` | - | 保留，代价公布 |
| `.../probe/libm_caller_probe.c` 与 `libm_versions.map` | 调试必备 | 手动 | - | 保留 |
| `.../probe/fast_rounding.c` | 冻结产物 | - | 被否的优化，分诊规则的范例 | 保留，绝不当作工具 |
| `.../probe/fast_rounding_check.c` | 调试必备 | 手动 | - | 保留 |
| `projects/spine-runtime/src/spine_runtime_project/a100_probe.cc` | 工具 | `scripts/run.sh` | - | 保留 |
| `.local/llvm.sh`、`.local/llama.cpp/{diag,test-dl-off,diagnostic-probe.patch}` | 临时 | - | 构建排查残留 | 该轮结束后经批准删除 |
| `.local/coremark-work/` | 冻结产物 | - | 各轮引用的实验记录 | 保留在 Git 之外 |

## 登记粒度

一行可以指向一个包目录（`foundation/src/labctl/`、`projects/coremark/src/coremark_project/`），
也可以指向单个文件（`experiments/onnx-ep-profile.py`）。包行覆盖其内部所有模块与测试：它们
是同一个工具、同一个角色，逐个列出反而会掩盖这一点。**尚未审计的代码按原样登记，而不是
省略。**

## 已声明的缺口

| 缺口 | 状态 |
|---|---|
| `projects/onnxruntime` 没有 `scripts/` 入口，也没有 `_project` 包 | 已声明：provider 闭源且自动化尚未编写；登记表明说，而不是假装项目完整 |
| `projects/onnxruntime` 与 `projects/spine-runtime` 没有项目局部 `AGENTS.md` | 开放：coremark 与 llama.cpp 已有 |
| 六个硬件仓库与三个新项目不在 `.repo/manifests/default.xml` 中 | 开放：它们的远端尚不存在，写进清单会破坏 `repo sync`；条目已作为评审材料写在各 README |

## 由此得出的规则

- 一个仓库暴露两个入口；它发布的其它一切都要经这两个入口触达。工作流需要第三个动词
  时，它成为子命令，而不是新脚本。
- 临时板卡访问只有一个工具（`board.sh`）；各 `run.sh` 内部的重复是**有意的自洽**，
  不是合并候选。
- 产出过封存证据的工具要冻结而不是删除，并在原处标明被谁取代。删除它会破坏那份证据
  再生成的能力。
- 临时项位于 Git 之外、登记在此，并在该轮结束时移除——按工作区契约，清理属于破坏性
  操作，需人工批准。
- 重叠靠**指明存活者**解决：结论复核用 `onnx-ep-analysis.py`，单包摘要用
  `package-summary.py`，llama.cpp 测量用 `bench.sh`。
