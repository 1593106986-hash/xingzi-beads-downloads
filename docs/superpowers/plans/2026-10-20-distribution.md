# 星子拼豆多端分发 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 保留现有创作体验，增加静态官网、本地迁移、PWA 和原生打包入口。

**Architecture:** 现有前端加独立分发面板，官网独立构建；共享版本协议；Tauri/Capacitor 只提供平台能力。

**Tech Stack:** React / TypeScript / Vite / IndexedDB / Web Crypto / Tauri 2 / Capacitor。

**Spec:** `docs/07-cross-platform-design.md`

## Global Constraints

- 保留 A1–G1、数据库 bead-studio v1、现有 bead-* 偏好键及所有核心算法和导航交互。
- 无账号、业务后端、广告、遥测；不上传任何图片或项目。
- 静态检查默认关闭，所有刷新/原生更新需用户触发。
- 不为未构建的原生包生成假下载、文件大小或校验和。

## Review Focus

- 密码错误、密文篡改、未来版本和关联断裂必须在写库前拒绝。
- 恢复备份不能覆盖同 ID 本地项目，必须包含原图、独立色板与偏好。
- 自动保存与导入/清空不能竞争并复活已删数据。
- 版本清单不允许 javascript:、跨域安装包或路径穿越；不缓存旧版本清单。
- 浏览器提示和更新不能抢占路线导航键、覆盖原有帮助或阻塞工作台。

### Task 1: 备份与版本协议

**Files:** `src/features/distribution/{backup,crypto,release}.ts`，`src/storage/db.ts`，`tests/distribution.test.ts`。

**Interfaces:** `encodeBackup(data,password?): Promise<string>` / `decodeBackup(text,password?): Promise<unknown>`；`createBackup(projectId?): Promise<Backup>` / `restoreBackup(raw): Promise<number>`；`parseRelease(raw): ReleaseManifest` / `detectPlatform(ua): Platform`。

- [ ] 写真实备份往返、错误密码/篡改、合并恢复、恶意清单链接和平台识别测试；运行确认缺少功能失败。
- [ ] 实现加密和校验、保存队列内事务恢复，旧项目 JSON 入口不变。
- [ ] 运行新测试及全套单元测试。

### Task 2: 应用分发面板与 PWA

**Files:** `src/features/distribution/{DistributionPanel,InstallHint,runtime,platform}.tsx/ts/css`，`src/app/App.tsx`，`src/main.tsx`，`vite.config.ts`，`index.html`。

**Interfaces:** `useDistribution` 状态；`startDistribution()` 生命周期；`saveFile(blob,name)` 原生/浏览器分流。

- [ ] 添加安装/关于/数据入口；首次提示记忆；用户确认更新、备份导入/导出和清空。
- [ ] 保留核心 UI，仅替换品牌文案；安全区、桌面保存/导入快捷键与 Android 返回处理。
- [ ] 构建应用，执行现有浏览器解释/导航回归和新增离线/备份检查。

### Task 3: 官网与发布产物

**Files:** `website/`，`vite.site.config.ts`，`public/version.json`，`scripts/{assemble-site,release-manifest}.mjs`。

- [ ] 官网使用真实截图和原图标，静态锚点内容，系统主题/手动主题，推荐平台和真实清单下载。
- [ ] 组合输出官网+app，相对路径；生成真实 SHA256、体积、平台与签名状态。
- [ ] 运行官网构建及手机/桌面浏览器验收，下载校验和实测。

### Task 4: 原生工程与构建文档

**Files:** `src-tauri/`，`capacitor.config.ts`，`android/`，`.github/workflows/`，`docs/08-distribution-guide.md`，`docs/09-distribution-verification.md`，`README.md`。

- [ ] 安装明确版本的 Tauri/Capacitor 依赖并生成平台工程；配置最小权限与可选签名 updater。
- [ ] 提供 Windows/macOS/Linux、Android/iOS 构建 CI 和签名配置；秘密只用环境变量。
- [ ] 运行当前主机可执行的原生配置检查和构建，记录工具缺失与未验证项。
- [ ] 运行完整测试/构建/浏览器验收后进行一次独立审查，修复影响目标的缺陷，保存实际结果。

此目录不是独立 Git 仓库（向上意外发现 C:/ 仓库且所有权不匹配），不更改全局 Git safe.directory，不自动提交到系统根仓库。
