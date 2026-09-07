[English](repository-layout.md)

# 仓库目录规范

## Repo 工作区根目录

`k3Vrp5` 根目录是 Google `repo` 工作区，不是 Git 仓库。只有 `.repo/` 和由
manifest 生成的根文件直接位于该层。

```text
k3Vrp5/
├── .repo/
├── AGENTS.md
├── AGENTS.zh-CN.md
├── governance/
├── foundation/
├── projects/
│   └── coremark/
└── reports/                    可选；默认同步不存在
```

manifest 从 governance 复制根协作文件对。凭据、缓存、构建目录、原始候选和报告
都不归工作区根目录所有。

## Manifest 仓库

```text
manifest/
├── README.md
├── README.zh-CN.md
├── default.xml
└── releases/
    └── coremark-baseline.xml
```

`default.xml` 可以跟踪经过审核的开发分支。`releases/` 下的文件固定准确项目
commit ID。reports 使用 `reports,notdefault` 分组。

## Governance 仓库

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

governance 只包含规范和评审记录，不包含设备运行代码、评测补丁、报告或原始证据。

## Foundation 仓库

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

foundation 只拥有公共环境、设备、遥测、统计、证据、文档和报告契约行为。评测专用
解析器或优化必须属于相应项目仓库。

## 项目仓库

每个项目都是一个 Git 仓库。CoreMark 不再嵌套于通用 CPU 项目。

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

`scripts/` 目录恰好包含两个 Shell 文件。复现用法归项目 README 文档对所有。冗余
README、候选矩阵脚本、历史诊断和按实现模块机械创建测试文件均被禁止，除非它们
分别通过代码必要性门禁。

## Reports 仓库

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

每个项目只有一份逻辑报告。不得创建周期、日期、`latest`、`related` 或版本目录。
Git 历史提供继承关系。已接受的精简证据可以按 digest 跟踪；完整大型日志继续位于
外部归档，并通过稳定位置、SHA-256 和字节数引用。

## 仅本地数据

凭据、私有清单、恢复 bundle、下载的套件和工具链、构建目录、设备同步区及候选
原始证据，均位于所有受管 Git 仓库之外。其位置通过操作员选择的绝对路径传给项目
公开脚本。如果该根目录缺失、为相对路径、位于项目 Git tree 内部，或复用已有运行
目录，正式运行必须失败关闭。

## 命名与更新规则

- Git 仓库和项目 ID 使用稳定小写 kebab-case。
- 项目自有英文 Markdown 为 `name.md`，中文为 `name.zh-CN.md`。
- 禁止 `.en.md` 后缀。
- 报告文件名稳定并原位更新。
- Raw run ID 使用 UTC 且只允许追加；它不是报告文件名。
- 补丁使用四位有序前缀和 upstream 风格主题。
- 编译链、源码版本、运行时 profile 和证据使用准确版本与 digest 进行内容锁定。

## 执行方式

尽可能使用已有官方工具：使用 XML 解析和 `repo manifest` 检查工作区组成，使用 Git
检查历史和 ignore 边界，使用上游测试验证 benchmark 正确性，使用 ShellCheck 检查
公开 Shell 入口，并使用最少 foundation 验证器检查跨仓库证据契约。不得仅为检查本
文档而建立平行的 layout 框架。
