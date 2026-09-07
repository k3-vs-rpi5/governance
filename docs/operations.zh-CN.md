[English](operations.md)

# 实验室操作手册

## 本地准备

使用专用虚拟环境。目标设备缺少的通用依赖必须通过 APT 安装，并记录精确包版本。
不得静默使用 pip、user site、主机编译器或未声明的厂商仓库来满足可复现要求。

```bash
python3 -m venv foundation/.venv
foundation/.venv/bin/python -m pip install -e 'foundation[dev]'
foundation/.venv/bin/python -m pytest foundation/tests
PYTHONPATH=foundation/src foundation/.venv/bin/python -m labctl.cli docs-check \
  --root governance --catalog governance/config/documentation.yaml
```

设备地址、账号、密码和可选 identity-file 路径只保存在 `.device-credentials`。
该文件必须被 Git 忽略、权限为 `0600`、永不打印，并且只由操作员控制的 shell 加载。

## 已安装 OS 与设备验收

树莓派 5 与 K3 Pico 统一使用稳定 Type-C 供电和有线网络。本基线不构建或分发自制
烧录镜像。当前 Debian 和 Bianbu 已安装系统由环境 manifest、运行时 profile、
软件包/编译链哈希、服务与挂载摘要及重启证据共同锁定。任何重新烧录、OS 升级或
换板后，都必须重新采集 SSH probe 并通过这些门禁，之后才能接受结果。

不得只为限制核心数而重做镜像。如果 OS 能可靠暴露所需核心，就把进程和中断绑定
到声明的单核或四核 CPU 集合。更换 K3 板卡时建立新的设备身份和验收记录；重新验收
后仍复用逻辑 `k3-compatible` 与 `k3-development` profile。

## 运行时齐平

三条运行时轨道都是不可变输入：

1. `rpi-native` 使用树莓派 Debian 基线及其 APT 编译器；
2. `k3-compatible` 使用锁定的 Debian riscv64 rootfs、编译器、loader 和库；
3. `k3-development` 使用 Bianbu 原生运行时与声明的较新编译器。

严格齐平时，编译器与运行时都来自 compatible rootfs。Bianbu 高版本编译器配低
sysroot 只能作为补充结果。执行前检查 ELF 架构、interpreter、所需 symbol 版本、
sysroot 路径、编译器可执行文件摘要、linker、libc 和 wrapper 摘要。发生主机库泄漏
或缺少包锁时必须 fail closed。

临时本地 APT 仓库必须同时重定向 `sourcelist` 与 `sourceparts`；不得覆盖系统源
配置，也不得隐式读取已有第三方源。Python 构建依赖必须清除 `PYTHONPATH` 和
`PYTHONHOME`、禁用 user site，并记录实际导入模块的路径和摘要。

## 计时运行准备

每次正式项目运行前：

1. 验证源码、补丁 manifest、编译链、运行时 profile 和二进制摘要；
2. 验证有线网络、CPU 集合、governor、throttling、swap 和活动服务；
3. 采样配置/实际频率、温度、可见内存和存储状态；
4. 性能采样前先运行官方或代码自带正确性套件；
5. 源码下载、包安装、编译和存储 I/O 必须位于计时区间之外；
6. 在所有受管 Git 仓库之外的显式本地证据根目录下，封存日志、遥测、manifest、
   脚本副本和校验和。

当前 K3 UFS 2.2 与树莓派 SD 卡不得视为相同存储。计时前准备数据集；缓存必须按同一声明
策略预热或清除；任何不可避免的 I/O 都要单独报告，不得计入 CPU 结论。

当四核树莓派无法获得相同控制时，default parity 不得选择性停止 service，也不得
把 K3 后台任务隔离到额外核心。每条轨道应锁定活动 service 与 mount 摘要，记录
interrupt 和 swap 活动，并在每个 session 前通过同一项预注册 idle stability gate。
package manager、update job 和其他已知噪声任务必须停止。未通过 idle gate 的 session
判为 invalid 并重跑；threshold 与 observation window 必须在测量前写入项目 manifest。
只有单独标记的 `best-achievable` 结果才允许 K3 专用 service 或 cpuset 隔离，
`default-parity` 禁止使用。

## 项目执行

每个公开项目只有两个 Shell 入口：

```bash
projects/coremark/scripts/build.sh --help
projects/coremark/scripts/run.sh --help
```

quick 模式只用于诊断，不可进入报告。formal 模式必须使用锁定矩阵、足够时长、预定义
预热/session/样本、完整遥测和只追加 raw 输出。失败必须记录为 failed 或 blocked；
操作员不得修改 manifest 把它变成 passed。

### 远程长任务

SSH 前台进程不能作为正式运行的证据安全执行器。设备上的每个长任务必须通过主机级
托管服务启动：服务应能跨越 SSH 断线，以声明的非 root 设备账号运行，并保留
`Result`、`ExecMainCode`、`ExecMainStatus` 和 journal 记录。只有确认账号已启用
linger 时才可使用用户服务；否则使用显式声明 `User` 与 `WorkingDirectory` 的系统级
transient service。

启动前和完成后都要记录 boot ID。接受结果必须同时满足：boot ID 未变化、托管服务
结果及退出状态成功、样本数精确匹配、项目验证门禁通过且 checksum 闭包有效。SSH
断线本身不会使托管运行失效；设备重启、服务被停止、样本缺失或 manifest 不完整都
必须判定为无效。无效证据应在本地保留，下一次尝试前重新执行设备资格检查。托管服务
只提高交付可靠性，不得改变 benchmark 的 affinity、priority、环境或计时策略。

## 验证与发布

只执行组件适用的本地契约、Shell 语法、ShellCheck、包构建、上游、设备、正式运行
和干净工作区门禁。使用全新工作区验证 release manifest：

```bash
repo init -u <approved-manifest-url> -m releases/coremark-baseline.xml
repo sync
test ! -e reports
```

以上尖括号内容是操作员提供的已批准 URL，不是受跟踪的 manifest 内容。reports
必须通过显式选择 reports 分组同步。禁止自动推送。必须在审阅干净工作区结果后，
明确批准远端 URL、目标分支和 refspec。
