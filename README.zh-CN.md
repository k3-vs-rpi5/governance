[English](README.md)

# K3 与树莓派 5 可复现性能实验室

## 目标

本 Google `repo` 工作区建立公开、可复现的验证体系，用于对比进迭时空 K3 Pico
与树莓派 5，并优化 K3 软件。第一个独立项目是 CoreMark。governance、公共
foundation、项目和报告具有独立 Git 历史。程序只采集性能；功耗由外部设备在相同
且有记录的平台控制条件下测量。

## 固定对比模型

每份报告严格分开三条轨道：`rpi-native`、`k3-compatible` 和
`k3-development`。每个优化项目同时发布 `default-parity` 与
`best-achievable`；K3 优化结果不得替代统一参数的默认基线。报告必须记录 CPU
亲和性、频率、温度、内存、运行时、精确编译链、存储、有线网络、命令和验证状态。

## 仓库入口

- [工作区架构](docs/repo-workspace-architecture.zh-CN.md)规定 Google `repo` 拓扑和
  迁移边界。
- [仓库契约](docs/repository-contract.zh-CN.md)规定强制政策。
- [项目要求](docs/project-requirements.zh-CN.md)规定测试周期。
- [目录规范](docs/repository-layout.zh-CN.md)规定唯一规范路径。
- [操作手册](docs/operations.zh-CN.md)规定本地与设备执行方法。
- 同步 foundation 仓库中的平台基线区分厂商规格、板卡清单和实验室控制条件。

## 权威公开工作区

五个仓库作为公开同级仓库位于
[`k3-vs-rpi5`](https://github.com/k3-vs-rpi5)：`governance`、`foundation`、
`coremark`、`reports` 和 `manifests`。全新开发工作区从
`https://github.com/k3-vs-rpi5/manifests.git` 初始化；default manifest 不包含
reports。可以保留由操作人员管理的私有 Gitea 镜像，但它们不是权威 remote。

## 本地验证

建立隔离的 foundation 开发环境，只执行其保留的契约测试：

```bash
python3 -m venv foundation/.venv
foundation/.venv/bin/python -m pip install -e 'foundation[dev]'
foundation/.venv/bin/python -m pytest foundation/tests
foundation/.venv/bin/python -m ruff check foundation/src foundation/tests
projects/coremark/scripts/build.sh --help
projects/coremark/scripts/run.sh --help
```

项目对外执行只允许使用项目所属的 `scripts/build.sh` 和 `scripts/run.sh`。缺失的
构建依赖和编译器包必须通过 APT 按声明版本安装；下载的源码、编译器、凭据、原始
运行和功耗图片均保留在本地，除非契约明确允许该产物公开。

## 发布状态

公开源码仓库不等于批准性能结果。公开结果仍必须通过组件与设备门禁、不可变
release manifest、独立报告审核和证据身份闭环。GitHub `main` 是已批准的权威源码
ref；报告晋升仍是独立的人工决策。
