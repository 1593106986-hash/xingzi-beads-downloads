# SDD ledger — plan: docs/superpowers/plans/2026-10-20-distribution.md

## 2026-10-20 原生发布更新

已安装并使用 Windows 与 Android 工具链，生成 Windows NSIS/便携 EXE 以及正式签名 Android APK/AAB。Windows 已实际安装、启动和卸载；真实下载文件已核对 SHA256。官网已部署到 <https://xinzi.work/>。当前验收详情以 `docs/09-distribution-verification.md` 为准。

下方保留最初阶段的记录，其中工具链缺失、没有原生包等说明已由本次发布取代。Android 真机权限/相机等验收尚需具体设备。

## 最初阶段记录

- 用户已要求方案后直接生成代码及文档，且尽量减少人为操作；按现有前端 Extension 模式执行。
- Ruling: 不创建 worktree/提交 — 此目录没有独立 .git，向上发现系统 C:/ 仓库且所有权不符；不触碰系统仓库配置。
- Builder check: 明确备份格式、保存锁、相机降级和 UA 架构局限；指标只报告实测，原生工程与已验证安装区分。
- Task 1: completed — 加密备份/恢复、版本清单验证与单元测试已实现。
- Task 2: completed — 应用安装/关于/数据入口、PWA 提示、跨端备份与更新确认已实现。
- Task 3: completed — 静态官网、多端推荐、发布清单生成、校验和与桌面/手机浏览器验收已实现。
- Task 4: completed for source/configuration — Tauri、Capacitor、CI 模板和签名/部署指南已提供；未安装平台 SDK 的原生包没有声称已构建。
- Verification: `npm test` 59/59; `npm run build:release`; distribution, app E2E, feature E2E, release-manifest checks; `npm audit` 0 vulnerabilities.
- Remaining release gates: install Rust/Cargo for desktop builds; Java/Android SDK for Android; macOS/Xcode and Apple Team for iOS; supply release signing identities and verify on target hardware.
