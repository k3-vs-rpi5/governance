[English](project-requirements.md)

# 项目要求

## 项目仓库模型

每个获准评测都是由 Google `repo` manifest 选择的独立 Git 仓库。CoreMark 的
项目 ID 为 `coremark`，不存在上层 `cpu-foundation` 项目。公共设备、运行时、
环境、遥测和证据验证属于 foundation，项目不得复制。正式结果只认可由固定 revision
组成的完整工作区。

## 必须使用三条平台轨道

每个优化或验证项目都必须运行三条轨道；只有技术上不适用时才允许明确标记：
`rpi-native`、`k3-compatible` 和 `k3-development`。报告不得合并两个 K3
环境。树莓派是脚本验证本源；K3 Bianbu 是主要开发环境。

## 必须使用两种结果模式

每个项目分开发布 `default-parity` 与 `best-achievable`。默认齐平在所有轨道使用
相同官方源码、移植层、参数、种子、时长、CPU 集合、遥测契约和优化等级。
best achievable 可使用经过审阅的 K3 ISA/编译器/运行时/补丁优化。最佳结果不得
替代默认结果。

## 完整基础信息

每份结果报告必须记录：

1. 板卡/CPU 型号、架构、拓扑、在线核心、CPU 集合和可见内存；
2. 配置频率、采样平均频率、采样间隔/方法和 governor；
3. 平均温度、传感器路径、采样方法、throttling、swap 和活动服务；
4. OS 版本、内核、loader、libc、编译器、linker、完整 flags 和可执行文件摘要；
5. 运行时 profile 摘要、rootfs/sysroot/wrapper 身份和 ABI 兼容结果；
6. 内存带宽套件/方法/结果，以及 cache、branch、perf 和辅助 counter；
7. 实际 UFS/eMMC/SD 存储类别、缓存策略、I/O 排除、有线网络和 Type-C 供电控制；
8. 官方套件/版本/源码/校验和、命令、预热、session、样本和统计方法；
9. raw-run ID、验证结论、偏差、限制和复核人。

未知项必须写为 `not-recorded` 并说明原因。报告必须分别提供内容完整的英文
Markdown 与中文 Markdown；只有翻译后的标题而没有正文属于不合规。日志采用 UTC
命名 `YYYYMMDDTHHMMSSZ-<track>-<suite>-<stage>.log`，raw run 只允许追加。

## 稳定报告策略

独立 reports 仓库为每个项目只保留一个稳定目录：`<project-id>/`。重复测试在
操作员选择的证据根下建立新的只追加候选，绝不直接更新报告。经过人工审核后，
已接受的精简证据及其索引才晋升到 reports 仓库，同时原位更新稳定报告。禁止日期
报告目录、周期子目录和多个当前报告；Git 历史就是继承记录。

## 固定对外交付

每个公开项目必须且只能交付：

1. 项目仓库中的 `scripts/build.sh`，唯一公开构建入口；
2. 项目仓库中的 `scripts/run.sh`，唯一公开运行/测试入口；
3. `patches/`，其中含中英文说明、有序补丁和 `manifest.yaml`；
4. 包含完整复现步骤的项目 README 文档对；
5. 独立 reports 仓库中的 `<project-id>/report.md`、`report.zh-CN.md`、
   `report.yaml` 和 `evidence/index.yaml`。

构建入口必须拉取锁定的官方/上游源码、验证完整性、通过 APT 安装缺失的声明包、
验证精确编译链、以零 fuzz 应用 manifest 补丁序列，并写出 build manifest。
Python 构建模块必须清除 `PYTHONPATH`/`PYTHONHOME`、禁用 user site，并记录 APT
归属/路径/版本/SHA256。临时编译器 APT 仓库必须同时重定向 `sourcelist` 与
`sourceparts`，不得修改系统源。

运行入口必须重新验证构建/二进制身份，先执行官方或代码自带正确性套件，再执行
预注册矩阵、采集遥测，并在所有受管 Git 仓库之外的显式本地证据根目录封存 raw
输出。quick 模式不可进入报告；formal 模式只是已接受报告的可能输入，报告晋升仍是
独立的人工审核操作。

## 代码必要性门禁

每次新建源码文件、模块、脚本、helper、测试、fixture 或 framework 之前，作者必须
先停下来问一句：**我写的代码，是否有必要？** 默认答案为“没有必要”。只有用一句
必要性说明证明同时满足以下条件，才允许新增代码：

1. 它由已批准的公开产物或长期契约直接要求；
2. 现有仓库代码、官方/上游工具或测试、配置、文档审阅或已保存证据均未覆盖；
3. 它是最小且可长期维护的实现，引入的总复杂度不超过它消除的风险；
4. 它具有确定性验证方法、明确 owner 和删除条件。

答案不确定时就不写。优先复用、删除、配置、文档、官方套件或更简单的现有路径。
测试同样属于代码：计划中列过、测试数量、覆盖率目标、形式对称、假想的未来复用或
一次性迁移，都不能单独证明新增代码有必要。每个新增代码文件或职责内聚的测试组，
只需在工作计划或审阅记录中写一句必要性说明，不得在代码中散布形式化注释。已经
无法通过该门禁的代码，必须与它唯一保护的行为一起删除。

## 测试来源原则

有代码自带测试时必须优先使用。CoreMark 等标准分数必须使用官方套件，不得自创
替代测试。上游测试缺失或不充分时，补充测试必须经过审阅，具有已记录 oracle、
确定性 fixture、边界/负向/对抗用例、失败路径覆盖和独立复现。报告必须清楚标记
补充结果，绝不能把它当成官方分数。

## 本地测试判定规则

只有针对长期存在、可确定性判定且由本仓库负责的契约或回归，才新增本地自动化
测试：公开接口与 schema、安全或 fail-closed 边界、细微的解析/统计/身份行为、
仓库集成或隔离，以及与硬件无关的验收门禁。在最低权威层只写一个最小测试，等价
用例采用参数化。

不得重复官方或上游测试。不得为文案、一次性迁移、静态硬件事实、实时读数、已经
过 schema 检查的简单声明值，或低层已经覆盖的不变量编写单元测试。这些事项应
分别进入文档审阅、合规 ledger、准入证据、官方套件或稳定报告。测试数量和覆盖率
百分比都不是交付目标。完整判定与删除规则以
`docs/repository-contract.md` 和 `docs/repository-contract.zh-CN.md` 配对文件为准。

## CPU 与辅助套件

内存带宽遵循上游 STREAM；cache、syscall、thread、cryptography、compression、
allocator 和库测试使用相应上游或代码自带套件。每项结果都必须说明 workload、
metric、方向、单位、CPU 集合、方法、限制和发布条款。前期 CPU 基线使用免费套件；
授权 SPEC CPU 内容只保存在本地，不是首个免费套件基线的前置条件。

## CoreMark 官方分数轨道

CoreMark 1.0 固定为
`b56889a7bd9d5624fbeff2a73ba69878e6a9c146`，必须通过 `make check`，所有受
`coremark.md5` 控制的 workload 文件保持不变。官方分数只允许修改
`core_portme*`、编译/构建/链接选项和 ISA 选择。上游 main
`1f483d5b8316753a742cbf5590caf5bd0a4e4777` 的干净检出无法通过自身完整性检查，
因此被排除。workload 源码实验必须标记为 `diagnostic-only`。

可报告轨道使用相同的 `formal_runner_sha256` 和平台状态采集器身份。每个 raw 包
都必须包含脚本、编译器/wrapper 摘要、遥测和 `SHA256SUMS`。K3 研究目标是按实测
平均频率达到 `7.7+ CoreMark/MHz`。quick 候选必须至少超过锁定最佳值 `1%`，才
允许进入 `3 sessions x 30 samples`。

## K3 齐平编译器与运行时

主要 `k3-compatible` 齐平结果的编译器和运行时都来自锁定的 Debian 13 riscv64
rootfs。GCC 14、binutils、glibc、loader 和包版本，在两个架构均发布相同版本时
与树莓派 Debian 13 匹配。Bianbu 高版本编译器配低 sysroot 只能作为单独标记的
补充结果，绝不能称为齐平。

必须验证 wrapper 或显式 `--sysroot`、可移植 ISA flags、loader/library 路径、
包锁和所需 ABI symbols。主机库泄漏、未声明编译器回退、要求更新的 glibc symbol，
或缺少 rootfs/compiler 摘要时必须 fail closed。
