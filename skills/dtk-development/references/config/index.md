# DConfig 使用入口

## 触发关键词

本文档适用于以下任务场景：

**自然语言描述：**
- 应用配置/持久化存储
- 跨应用/跨进程配置共享
- 系统级配置/用户级配置
- 配置热加载/配置调试

**API 接口描述：**
- DConfig / DSettings
- dde-dconfig / dconfig2cpp

---

## 概述

DConfig 是 DTK 提供的跨进程配置管理方案，支持应用配置的持久化存储、
跨应用共享和 OEM 覆盖。客户端由 dtkcore 提供，服务端由
`dde-app-services` 中的 `dde-dconfig-daemon` 提供。

## 适用场景

- 应用需要持久化配置
- 多个应用需要共享配置
- 需要区分用户配置或系统级配置
- OEM 需要预置默认配置

## 模块约束

- 区分 `appId` 与配置 ID（`name`）：`appId` 可由 desktop 文件标识或
  `DSG_APP_ID` 声明，并与 `dde-application-manager` 的识别一致；配置 ID
  使用 meta 文件 basename，并采用稳定的倒置域名命名。
- 保持 appId 目录、配置 ID 文件名、C++/DBus 参数和 override 路径一致。
  排查缓存前先确认 daemon 版本和实际状态目录，不硬编码历史路径。

## 使用前置决策

所有 DConfig 任务先完成以下决策，再读取具体用法文档。

### 1. 固定配置身份

先从现有 desktop 文件、meta 文件和打包内容中确认标识；创建新配置时再按
[命名规范](concepts.md#2-标识与命名规范)设计。后续文档统一使用以下变量，
不重新推导或改名：

| 变量 | 固定含义 |
|------|----------|
| `appId` | desktop 文件 basename 或 `DSG_APP_ID` 声明；与应用管理器识别一致 |
| `configId` | meta 文件 basename，不含 `.json`；生产配置使用稳定倒置域名 |
| `subpath` | 可选运行时配置层级，不属于 `appId` 或 `configId` |

例如：

```text
desktop 文件：deepin-editor.desktop
appId：deepin-editor
meta 文件：org.deepin.editor.json
configId：org.deepin.editor
```

### 2. 选择访问方式

先确定业务代码使用 C++ 还是 DBus，再读取对应文档：

| 场景 | 访问方式 | 后续文档 |
|------|----------|----------|
| DTK C++ 应用读写配置 | `DConfig`；需要类型安全封装时使用 `dconfig2cpp` | [dconfig-cpp.md](dconfig-cpp.md) |
| 脚本或非 C++ 程序直接访问服务 | DBus | [dconfig-dbus.md](dconfig-dbus.md) |

同一业务功能优先使用一条稳定访问链路，不在应用代码中同时维护 C++ 与裸 DBus
两套配置读写实现。只有检查值、热加载或排查缓存时，才额外读取
[dconfig-debug.md](dconfig-debug.md)，调试命令不作为应用的配置访问实现。

### 3. 判断是否安装配置文件

- 应用是该配置的提供者时，安装 `{configId}.json` meta 文件到
  `/usr/share/dsg/configs/{appId}/`；构建安装示例见
  [concepts.md](concepts.md#cmake-安装)。
- 只读取目标环境已经提供的配置时，不重复安装 meta 文件；先确认目标环境中
  已有的 `appId`、`configId`、权限和公开接口。
- 只有 OEM 或管理员需要覆盖默认值时才安装 override 到
  `/etc/dsg/configs/overrides/{appId}/{configId}/`。普通用户值由 daemon
  持久化，应用不安装或手动写缓存文件。
- 应用无关配置不使用应用目录，先确认确实需要所有应用共享，再按
  [应用无关配置](concepts.md#7-应用无关配置)处理。

## 相关项目

| 项目 | 仓库 | 说明 |
|------|------|------|
| dtkcore | [linuxdeepin/dtkcore](https://github.com/linuxdeepin/dtkcore) | DConfig 客户端 API、dconfig2cpp 工具 |
| dde-app-services | [linuxdeepin/dde-app-services](https://github.com/linuxdeepin/dde-app-services) | dde-dconfig-daemon 服务、dde-dconfig CLI |
