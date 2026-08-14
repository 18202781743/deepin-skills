# Eval: 区分 DConfig appId 与配置 ID

## 任务

一个应用的 desktop 文件是 `deepin-editor.desktop`，DConfig meta 文件是
`org.deepin.editor.json`。说明 appId、配置 ID、meta 与 override 安装路径，
说明 `DSG_APP_ID` 的作用，并给出使用默认 appId 和显式 appId 创建 DConfig
的方式。

## 期望输出

回答应明确：

1. appId 是 `deepin-editor`；可来自 desktop 文件 basename，也可通过
   `DSG_APP_ID` 声明，但必须与应用管理器识别的身份保持一致。
2. 配置 ID（`name`）是 `org.deepin.editor`，来自 meta 文件 basename。
3. meta 安装到
   `/usr/share/dsg/configs/deepin-editor/org.deepin.editor.json`。
4. override 安装到
   `/etc/dsg/configs/overrides/deepin-editor/org.deepin.editor/`。
5. 默认构造传入配置 ID，appId 由 `DSGApplication::id()` 获取。
6. 只有确需显式指定时才调用
   `DConfig::create("deepin-editor", "org.deepin.editor", ...)`。

## 验证要点

- [ ] 没有把 appId 与配置 ID 当成同一概念
- [ ] 说明 desktop 文件标识与 `DSG_APP_ID` 都可用于声明 appId
- [ ] appId 与 `dde-application-manager` 识别的应用身份一致
- [ ] 配置 ID 使用稳定的倒置域名
- [ ] C++ 参数顺序与安装路径正确
- [ ] subpath 没有被当成配置 ID 的一部分
