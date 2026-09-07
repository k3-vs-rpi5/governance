[English](repository-consolidation-plan.md)

# Repo 工作区迁移实施计划

> **Agentic 执行者要求：** 必须使用 `superpowers:subagent-driven-development`
> 或 `superpowers:executing-plans` 逐任务实施本计划。步骤使用复选框（`- [ ]`）
> 跟踪。

**目标：** 将当前单体仓库转换为经过验证的 Google `repo` 工作区，使 governance、
foundation、CoreMark 和 reports 拥有独立 Git 历史，同时默认迁移不包含 reports。

**架构：** 当前工作树是恢复源，不是最终仓库边界。先在原位规范化内容，再拆分为
候选组件目录；验证后分别建立一次干净提交，最后通过使用同级相对远端的 manifest
同步出全新工作区。reports 项目被固定版本，但归入 `reports,notdefault`。

**技术栈：** Google `repo`、Git、Python 3.11+、PyYAML 6.x、pytest 8.x、
Ruff、POSIX Shell、ShellCheck、APT、CoreMark 1.0、Debian 13 riscv64 rootfs、
Bianbu 4.0.4，以及固定的 GCC/Clang 编译链。

**设计：** [repo-workspace-architecture.zh-CN.md](repo-workspace-architecture.zh-CN.md)

## 全局约束

- 不推送任何仓库。
- 在生成并验证恢复资料前，不重写或丢弃当前工作树、index、被忽略证据或权限为
  `0600` 的 `.device-credentials`。
- 每个目标仓库只有在其源目录通过适用的基本检查后，才能创建第一次提交。
- 默认同步的工作区不得包含 `reports/`。
- 最终 manifest 不得包含机器相关的绝对 URL 或占位符。公共 remote 使用同级
  相对 fetch URL；本地绝对 URL 只允许出现在被忽略的迁移证据中。
- CoreMark 项目 ID 为 `coremark`；目标架构中不存在 `cpu-foundation` 项目。
- 每个项目只有一个 `build.sh`、一个 `run.sh` 和一份稳定逻辑报告；不得复制日期版
  报告。
- 树莓派 5 是脚本验证本源，K3 Bianbu 是主要开发环境。正式工作保留
  `rpi-native`、`k3-compatible`、`k3-development` 三轨，并分开
  `default-parity` 与 `best-achievable`。
- 优先使用官方或上游正确性套件。自有测试只保护长期自有契约；不设测试数量或
  覆盖率目标。
- 缺失软件包通过 APT 安装。编译器、链接器、libc、运行时、源码、参数和补丁身份
  必须明确记录。
- Markdown 必须具有完整中英文配对。当现有权威文档对可以承载内容时，不新增
  说明文档。
- 有意内容修改使用 `apply_patch`。审阅源目录和目标清单后，机械拆分可以使用
  `rsync`。

---

### 任务 1：冻结并清点已批准的源工作树

**文件：**

- 本地新建：`.local/repo-migration/recovery/pre-migration.bundle`
- 本地新建：`.local/repo-migration/recovery/status.txt`
- 本地新建：`.local/repo-migration/recovery/index.diff`
- 本地新建：`.local/repo-migration/recovery/worktree.diff`
- 本地新建：`.local/repo-migration/inventory/path-owners.tsv`

**接口：**

- 输入：`image-delivery-baseline` 分支、index、未暂存的 operations 更新、被忽略
  证据和已批准架构文档对。
- 输出：经过验证的恢复点和完整的路径到目标 owner 映射。

- [x] **步骤 1：记录身份且不读取秘密内容**

  在 `.local/repo-migration/recovery/` 下记录以下命令的输出：

  ```bash
  git rev-parse HEAD
  git status --short
  git diff --cached --binary
  git diff --binary
  git ls-files
  ```

  只记录 `.device-credentials` 的路径、权限和 ignore 状态，绝不记录其值。

- [x] **步骤 2：创建并验证恢复 bundle**

  执行以下命令并在本地保存验证输出：

  ```bash
  git bundle create .local/repo-migration/recovery/pre-migration.bundle --all
  git bundle verify .local/repo-migration/recovery/pre-migration.bundle
  ```

- [x] **步骤 3：记录可执行基线**

  执行 `python3 -m pytest -q`、`python3 -m ruff check src tests projects`、两个
  CoreMark 入口的 `sh -n`、ShellCheck，以及 CoreMark 补丁无模糊往返。将准确输出
  和退出状态保存在本地。

- [x] **步骤 4：对每个保留的受跟踪路径分类**

  写入 `path-owners.tsv`，每个路径只能归属 `manifest`、`governance`、
  `foundation`、`coremark`、`reports` 或 `drop` 之一。当前 CoreMark 代码和补丁
  归 coremark；通用运行时/环境逻辑归 foundation；规范归 governance；报告/证据
  归 reports；一次性旧整合机制归 drop。

- [x] **步骤 5：检查点**

  验证全部恢复文件均被忽略、bundle 可读，且采集期间没有修改 Git remote 或受
  跟踪文件。

### 任务 2：使 governance 成为唯一协作与规范权威

**文件：**

- 新建：`AGENTS.md`
- 新建：`AGENTS.zh-CN.md`
- 修改：`README.md`
- 修改：`README.zh-CN.md`
- 修改：`docs/design.md`
- 修改：`docs/design.zh-CN.md`
- 修改：`docs/operations.md`
- 修改：`docs/operations.zh-CN.md`
- 修改：`docs/project-lifecycle.md`
- 修改：`docs/project-lifecycle.zh-CN.md`
- 修改：`docs/project-requirements.md`
- 修改：`docs/project-requirements.zh-CN.md`
- 修改：`docs/repository-contract.md`
- 修改：`docs/repository-contract.zh-CN.md`
- 修改：`docs/repository-layout.md`
- 修改：`docs/repository-layout.zh-CN.md`
- 修改：`config/documentation.yaml`

**接口：**

- 输入：已批准架构和现有长期验证规则。
- 输出：不再包含 `cpu-foundation`、单体仓库报告路径、两提交历史或仅聊天授权假设
  的可迁移规范。

- [x] **步骤 1：编写根协作契约**

  新增简洁的 `AGENTS.md` 和完整中文版本。要求读取 governance/项目契约、执行
  代码必要性提问、控制变量验证、优先上游测试、先有证据再作结论；发布、破坏性
  操作、remote 和 push 必须人工批准；禁止泄露秘密。

- [x] **步骤 2：重写现有权威文档而不是新增指南**

  将已批准的 repo 拓扑、reports 分离、单一项目单一报告、三轨环境、子项目周期和
  最小测试规则合并进现有权威文档对，删除旧 `cpu-foundation` 和报告位于单体仓库
  的模型。

- [x] **步骤 3：更新文档目录**

  登记架构和 `AGENTS` 文档对；修改过的文档对递增 revision，并统一中文评审状态。
  不为规范正文新增自动化测试。

- [x] **步骤 4：验证 governance**

  执行 `git diff --check`、现有文档检查器、本地链接检查，以及旧规范字符串精确
  搜索。旧字符串只允许出现在架构文档的迁移影响章节或被忽略恢复证据中。

### 任务 3：收敛 foundation 实现与测试表面积

**文件：**

- 删除：`src/labctl/coremark.py`
- 删除：`src/labctl/conformance.py`
- 删除：`src/labctl/readiness.py`
- 删除：`src/labctl/release.py`
- 删除：`tests/test_coremark_summary.py`
- 删除：`tests/test_conformance.py`
- 删除：`tests/test_readiness.py`
- 删除：`tests/test_release.py`
- 修改：`src/labctl/cli.py`
- 修改：`src/labctl/documentation.py`
- 修改：`src/labctl/governance.py`
- 修改：`src/labctl/layout.py`
- 修改：`tests/test_cli.py`
- 修改：`tests/test_documentation.py`
- 修改：`tests/test_governance.py`
- 修改：`tests/test_layout.py`
- 修改：`pyproject.toml`

**接口：**

- 输入：新的分仓契约。
- 输出：作为公共环境/证据验证器的 `labctl`，其中不包含 CoreMark 专用解析或一次性
  单体仓库历史机制。

- [x] **步骤 1：为每个现有测试文件记录必要性处置**

  在实施评审记录中把每个保留文件映射到一项长期契约和删除条件。删除只服务于
  CoreMark、`cpu-foundation`、仓内 reports、一次性整合或旧历史形态的文件。不
  创建用于测试这份评审记录的 checker。

- [x] **步骤 2：先让最少的现有测试表达新边界并失败**

  修改 layout fixture，使其接受只有 `build.sh` 和 `run.sh` 的独立项目根目录；
  修改报告验证，使其接受仅包含 `coremark/report.md`、`report.zh-CN.md`、
  `report.yaml` 和 `evidence/index.yaml` 的 reports 仓库根目录。删除独立复现报告
  以及单体相对项目路径断言。

- [x] **步骤 3：运行聚焦测试并观察旧假设失败**

  执行 `pytest tests/test_layout.py tests/test_governance.py tests/test_cli.py -q`。
  预期失败必须指向旧报告/项目路径或被删除命令行为，而不是无关回归。

- [x] **步骤 4：删除旧模块并简化公共验证器**

  删除四个旧模块及其专用测试。从 CLI 删除 `coremark-summary`、干净历史 release
  scope 和硬编码 `cpu-foundation` readiness。环境、运行时、兼容性、遥测、统计、
  远程传输、套件锁和只追加证据行为，只有在现有测试保护已声明公共契约时才保留。

- [x] **步骤 5：合并重复契约用例**

  参数化同类文档/layout/报告失败，删除对文案、静态值、包装内部或已经由 schema/
  更低层验证的行为测试。不得用新框架替代已删除测试。

- [x] **步骤 6：验证 foundation**

  执行保留的 foundation 测试、Ruff 和包构建。确认源码和测试均不导入
  `labctl.coremark`、`labctl.conformance`、`labctl.readiness` 或
  `labctl.release`。

### 任务 4：把 CoreMark 提取为路径无关的独立项目

**文件：**

- 重命名：`projects/cpu-foundation/` 为 `projects/coremark/`
- 重命名：`projects/coremark/src/cpu_foundation/` 为
  `projects/coremark/src/coremark_project/`
- 新建：`projects/coremark/sources.lock.yaml`
- 新建：`projects/coremark/environments.lock.yaml`
- 修改：`projects/coremark/project.yaml`
- 修改：`projects/coremark/README.md`
- 修改：`projects/coremark/README.zh-CN.md`
- 修改：`projects/coremark/scripts/build.sh`
- 修改：`projects/coremark/scripts/run.sh`
- 修改：`projects/coremark/src/coremark_project/cli.py`
- 修改：`projects/coremark/src/coremark_project/delivery.py`
- 修改：`projects/coremark/src/coremark_project/manifests.py`
- 合并：`projects/coremark/tests/test_delivery_contract.py`
- 保留并简化：`projects/coremark/tests/test_patch_contract.py`
- 合并唯一内容后删除：`benchmarks/coremark-candidates.yaml`、
  `benchmarks/free-baseline*`、`benchmarks/suite-manifest.yaml`、冗余目录 README 和
  独立复现文档。

**接口：**

- 输出：项目 ID `coremark`、包 `coremark_project`、显式
  `COREMARK_EVIDENCE_ROOT`，以及恰好两个公开 Shell 入口。
- 输入：通过 manifest 工作区和固定环境 profile ID 使用 foundation，绝不依赖
  硬编码的单体仓库根目录。

- [x] **步骤 1：先编写两个最小项目契约测试**

  `test_delivery_contract.py` 只验证两个可执行入口、项目根发现、显式证据根约束、
  官方 CRC 与最短时间拒绝，以及 compatible rootfs 隔离。
  `test_patch_contract.py` 验证 manifest 身份和一次真实无模糊应用/撤销。删除打开
  仓内报告的测试。

- [x] **步骤 2：运行聚焦测试并观察失败**

  执行 `PYTHONPATH=projects/coremark/src:src pytest projects/coremark/tests -q`。
  预期失败应指出旧包名、旧绝对根目录推导或旧证据路径。

- [x] **步骤 3：重命名并精简项目内容**

  把所有规范性 `cpu-foundation` 身份替换为 `coremark`。将评测、优化、脚本和测试
  README 的唯一内容合并到项目 README 或补丁 README，再删除冗余文件。

- [x] **步骤 4：显式化路径与环境契约**

  从已安装的项目包推导 `PROJECT_ROOT`。要求绝对的 `COREMARK_EVIDENCE_ROOT`，且
  位于受跟踪项目内容之外。通过锁定 ID/digest 从同步工作区解析 foundation
  profile。profile 缺失、digest 不符、复用输出、主机库泄漏或未声明编译器回退
  必须失败关闭。

- [x] **步骤 5：修正执行顺序和元数据**

  先运行上游 `make check` 和官方正确性模式，再运行性能；执行声明的预热；记录
  run ID、boot ID、编译器/链接器/libc/binutils、完整参数、亲和性、采样频率、平均
  温度、内存/swap、存储、网络、节流、服务和失败状态。quick 结果不可进入报告，
  formal 结果只能追加。

- [x] **步骤 6：验证公开交付**

  执行项目 pytest、Ruff、`sh -n`、ShellCheck、上游 CoreMark `make check`、补丁
  应用/撤销和两个 `--help`。确认只有两个 `*.sh`，且脚本均不写报告。

### 任务 5：将 reports 重塑为单一项目、单一报告

**文件：**

- 移动：`reports/cpu-foundation/coremark-three-track/report.md` 到
  `reports/coremark/report.md`
- 移动：`reports/cpu-foundation/coremark-three-track/report.zh-CN.md` 到
  `reports/coremark/report.zh-CN.md`
- 移动并重写：`reports/cpu-foundation/coremark-three-track/report.yaml` 到
  `reports/coremark/report.yaml`
- 新建：`reports/coremark/evidence/index.yaml`
- 合并后删除：CoreMark `reproduction.md` 文档对及非项目治理/基础设施报告
- 修改：`reports/README.md`
- 修改：`reports/README.zh-CN.md`

**接口：**

- 输出：一份稳定 CoreMark 报告对和一个与代码迁移独立的内容寻址公开证据索引。

- [x] **步骤 1：移动前清点每项结论及其引用运行**

  将每项结论标记为 `verified-local`、`missing-evidence` 或 `blocked-retest`。绝不以
  名称相似的运行替代缺失运行。缺少证据的结论只作为限制说明，不能作为当前公开
  测量值。

- [x] **步骤 2：把复现内容合并进唯一报告和项目 README**

  公开结果解释和证据链接留在报告中；构建/运行步骤留在 CoreMark 项目 README。
  两个目标均已承载唯一内容后，删除独立 reproduction 文档对。

- [x] **步骤 3：建立机器可读报告身份**

  记录准确 manifest、governance、foundation、project、源码、工具链、补丁和
  证据身份。已接受精简证据按内容 digest 存放；外部完整日志记录位置、SHA-256、
  字节数和可用性。

- [x] **步骤 4：独立验证报告**

  使用 foundation 报告/证据验证器检查 reports-only fixture 和真实
  `reports/coremark`。确认报告不能导入项目源码，也不依赖当前单体仓库路径。

### 任务 6：定义 manifest 和组件拆分

**文件：**

- 新建：`manifests/README.md`
- 新建：`manifests/README.zh-CN.md`
- 新建：`manifests/default.xml`
- 组件提交后生成：`manifests/releases/coremark-baseline.xml`
- 本地新建：`.local/repo-migration/component-trees/`

**接口：**

- 输出：使用同级相对 remote 的 Google `repo` manifest，其中项目为
  `governance.git`、`foundation.git`、`coremark.git` 和 `reports.git`。

- [x] **步骤 1：机械组装经过审阅的组件目录**

  使用 `path-owners.tsv` 和 `rsync` 在被忽略迁移目录下建立 governance、
  foundation、coremark 和 reports 目录。将每个目标文件清单与 inventory 比较，
  拒绝重复或无 owner 路径。

- [x] **步骤 2：编写开发 manifest**

  使用一个 fetch 路径相对于 manifest 仓库 URL 的 remote。governance 检出到
  `governance`，foundation 到 `foundation`，CoreMark 到 `projects/coremark`，
  reports 到 `reports`。reports 分组为 `reports,notdefault`。将 governance 的
  `AGENTS.md` 和 `AGENTS.zh-CN.md` 复制到工作区根目录。

- [x] **步骤 3：不增加测试代码，直接验证 XML 和规范**

  使用 Python 标准库解析 XML，检查项目路径/分组，并确认不存在绝对本地 URL、
  占位符、重复路径，且 reports 不属于默认分组。

### 任务 7：验证、建立干净组件历史并演练迁移

**文件：**

- 本地新建：`.local/repo-migration/sources/<component>/`
- 本地新建：`.local/repo-migration/remotes/<component>.git`
- 本地新建：`.local/repo-migration/workspace/`
- 本地新建：`.local/repo-migration/evidence/`

**接口：**

- 输入：四个组件目录和 manifest 目录。
- 输出：五个本地独立 Git 历史，以及默认不含 reports 的全新同步工作区。

- [x] **步骤 1：第一次提交前运行基本检查**

  governance 通过文档/链接检查；foundation 通过保留测试、Ruff 和包构建；
  CoreMark 通过项目测试、ShellCheck、上游正确性和补丁重放；reports 通过双语与
  证据检查；manifest 通过 XML/规范检查。

- [x] **步骤 2：每个候选仓库建立一次干净根提交**

  初始化 `main`，只加入经过审核的组件内容，并执行以下检查：

  ```bash
  git status --short
  git diff --cached --check
  ```

  随后分别使用符合组件职责的消息提交一次。不配置或推送网络 remote。

- [x] **步骤 3：建立本地 bare 验证远端**

  把每个仓库作为同级 bare 仓库克隆到 `.local/repo-migration/remotes/`。最后初始化
  manifest 仓库，使其同级相对 remote 能解析所有组件。

- [x] **步骤 4：同步全新的默认工作区**

  在被忽略工作区中，使用本地 manifest bare 仓库执行 `repo init` 和 `repo sync`。
  验证 governance、foundation 和 `projects/coremark` commit 与 manifest 完全一致，
  且 `reports/` 不存在。

- [x] **步骤 5：验证协作规则发现和显式 reports 同步**

  确认根 `AGENTS.md` 与 governance 一致。显式选择 reports 分组，验证固定 reports
  commit 和 CoreMark 报告；随后移除该 checkout，并证明新的默认同步仍不包含报告。

- [x] **步骤 6：生成不可变 release manifest**

  导出固定 revision 的 manifest，保存为
  `manifests/releases/coremark-baseline.xml`，并在第二个干净工作区验证。只有当
  release 文件尚未纳入时才更新 manifest 仓库的第一次提交；不得仅为迁移诊断
  创建第二个公共历史提交。

### 任务 8：设备资格检查与完成门禁

**文件：**

- 本地新建：`<evidence-root>/foundation/<run-id>/`
- 本地新建：`<evidence-root>/coremark/<run-id>/`
- 仅在正式证据接受后更新：候选 reports 仓库中的 `coremark/report.md`、
  `coremark/report.zh-CN.md`、`coremark/report.yaml` 和
  `coremark/evidence/index.yaml`

**接口：**

- 输出：通过资格检查的三轨证据，或明确 blocked 状态，绝不输出虚假公开结论。

- [ ] **步骤 1：不暴露凭据地检查两台设备资格**

  加载被忽略的凭据文件，通过 SSH 连接，记录开发板身份、boot ID、OS/内核、有线
  网络、CPU 拓扑、单核/四核集合、频率/governor、温度、内存/swap、存储、服务、
  节流和编译链/运行时身份。不得打印密码值。

- [ ] **步骤 2：验证环境矩阵**

  验证 `rpi-native`、K3 锁定 compatible rootfs/chroot 和
  `k3-development`，生成明确偏差清单。存在未解决重要差异时，禁止使用严格齐平
  标签。

- [ ] **步骤 3：执行 CoreMark quick 资格检查**

  先在树莓派 5 验证两个公开脚本，再在树莓派 5 和 K3 两轨执行 default-parity，
  最后执行 K3 best-achievable。必须满足官方 CRC、至少十秒、声明的预热、稳定遥测
  和只追加封存证据。

- [ ] **步骤 4：决定是否允许 formal**

  如果设备复位、不可达、身份变化、运行时漂移或稳定性门禁失败，则写入 `blocked`
  证据并停止正式晋升；否则执行预注册正式 sessions 并验证每个数据包。

- [ ] **步骤 5：不推送地执行最终验证**

  重新执行全部组件检查和固定 release-manifest 同步。确认候选源仓库没有网络
  remote、默认工作区没有 reports、受跟踪内容没有秘密字面量，且没有缺乏证据的
  公开结论。准确报告剩余阻塞项，尤其是缺少最终 remote URL 或硬件不可用。

## 完成定义

当固定的本地 release manifest 可以重建干净默认 `repo` 工作区、reports 仓库可
单独审计、CoreMark 成为测试最小化的独立项目、三轨环境全部通过资格检查或明确
blocked，且没有推送或破坏性替换原工作区时，本次实施完成。
