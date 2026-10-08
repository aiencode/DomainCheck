# Pi v1.1.0 Linux x86_64 定点封堵版

## 基线

- 上游项目：`earendil-works/pi`
- 固定版本：`v1.1.0`
- 上游提交：`abe508e1b89912adde45528136c3221eb69acdd7`
- 官方源码归档 SHA-256：`63b17b48b855e36e64c5013523acd48131ffcfa90ae48fe2f3e6fa9fe3d0da32`
- 完整封堵补丁 SHA-256：`936fc7ee2a277cfc74997d373e0556bb8e9c08bcd81572b48c922fe9a8e79455`
- 目标平台：Linux x86_64 / x64 / AMD64

本版本只处理已经发现的数据外传与供应链信任问题，不删除或重构正常的 Provider、bash、extension、skill、subagent、session 等能力。

## 已封堵

### 1. 远程模型目录不能再改写请求目的地

- `pi.dev` 远程目录只能更新明确列入白名单的普通模型元数据。
- 内置模型的 Provider、模型类型、API、`baseUrl`、认证 Header 和兼容路由配置始终来自本地随程序发布的可信定义。
- 新模型 ID 只有在模型类型、API 和请求地址与本地已经发布的同 Provider 路由完全一致时才会被接受。
- 已存在的模型 ID 不允许被远程目录改变模型类型。
- `models-store.json` 中的历史缓存会在恢复前执行同样过滤，过滤结果会重新写回缓存。
- 用户自己在本地明确配置的自建 Provider、代理和自定义 `baseUrl` 不受影响。

### 2. Radius 分享必须显式选择

- `/share` 与 `/share gist`：创建用户自己 GitHub 账户中的私有 Gist。
- `/share radius`：明确上传完整 Session 到配置的 Radius 服务器。
- 已登录 Radius 不再被视为本次上传授权，也不会自动优先或回退到 Radius。

### 3. Bug 报告采用安全默认值

- 默认不附带 Transcript。
- 默认不让模型生成摘要。
- 默认导出本地 ZIP，而不是上传。
- 用户仍可逐项明确选择 Transcript、摘要和上传。

### 4. 安装遥测默认关闭

- `enableInstallTelemetry` 默认由开启改为关闭。
- 用户明确开启后原功能仍保留。
- `enableAnalytics` 继续默认关闭。

### 5. 自更新包身份与下载来源锁定

- 版本接口返回的 `packageName` 被忽略，自更新只能安装固定官方包 `@earendil-works/pi-coding-agent`。
- 远程版本必须是合法 SemVer，不能用 npm alias 等字符串改变包身份。
- Managed installer 在执行 `npm ci` 前校验根包身份、Pi 依赖、版本和 lockfile。
- Lockfile 中所有 tarball 必须精确指向 npm 官方 Registry 的对应包和版本，拒绝作者私服或任意第三方下载地址。
- 原有版本比较、更新流程与 `--ignore-scripts` 保留。

## 未改变的正常能力

- 用户主动使用 OpenAI、ChatGPT/OpenAI Codex、Anthropic、Google、GitHub 等官方服务。
- 用户自己配置的 API Provider、自建服务器、反向代理和自定义 `baseUrl`。
- Radius Provider 本身；仅禁止隐式使用及隐式上传 Session。
- bash、extension、skill、subagent、session、聊天记录与项目配置。

## 仍需 Linux 系统边界处理

Pi 的 bash 和扩展与启动 Pi 的 Linux 用户拥有相同权限。恶意提示、命令或第三方扩展仍可能访问该用户可访问的文件和网络。彻底限制这类通用能力需要容器、独立系统用户、网络命名空间或出站防火墙。本次没有为每条命令增加审批，也没有重写 Agent 架构。

## 构建与验证

构建流程从固定的官方 v1.1.0 源码归档开始，验证源码和补丁 SHA-256，随后执行：

- 远程模型目录回归测试
- `/share` 回归测试
- `/bug` 安全默认值测试
- Telemetry 默认值测试
- 版本接口与更新供应链测试
- 完整离线构建
- Linux x64 独立二进制构建
- `pi --version`、`pi --help` 与 Codemode 二进制冒烟测试

最终下载包包含 Linux x64 二进制、完整补丁、本说明、构建结果和 SHA256SUMS。

## 安装与数据保留

替换或卸载 Pi 主程序不会删除 `~/.pi/agent/`。原有聊天记录、技能、配置、扩展和子代理会继续保留。不要删除 `~/.pi/agent/` 或项目中的 `.pi/`。
