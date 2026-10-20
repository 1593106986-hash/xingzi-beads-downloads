# 豆序 Implementation Plan

> 执行方式：当前会话直接实现，用户已授权采用合理默认值并完成代码。先设计再实现，不再引入审批步骤。

**Goal:** 交付完整本地拼豆 PWA。
**Architecture:** 页面永久编号，领域引擎通过 Worker 与 UI 隔离，Zustand 统一历史状态，idb 持久化。
**Tech Stack:** React 18 / TypeScript / Vite / Zustand / Canvas2D / Worker / idb / Vitest / vite-plugin-pwa。
**Spec:** docs/01-product-and-architecture.md。

## Global Constraints

运行时无远程网络依赖；50/100 网格；只输出当前色板内颜色；重计算只在 Worker；固定 A1-G1 模块；用户图片不离开设备。

## Review Focus

透明图与空色板、尺寸变化后的历史与进度、旧 Worker 响应、棋盘格碎块性能、重新打开后的源图与色板恢复。

## Tasks

- [x] 1. 基础工程与领域契约。建立 package/config、src/shared/types.ts、页面目录。验证 npm install 和 TypeScript。
- [x] 2. 算法先验测试。tests/color.test.ts 校验 Sharma 值；tests/image.test.ts 校验 50/100 与闭包/锁定；tests/route.test.ts 校验覆盖与排序。先运行失败，再实现 src/core/color、image、route，确认通过。
- [x] 3. Worker/存储/状态。WorkerRequest → map/extract/route；ProjectRepository → save/load；历史 change/undo/redo。tests/storage.test.ts 验证事务，tests/history.test.ts 验证进度与编辑恢复。
- [x] 4. A1-G1 页面与可交互 Canvas。输入 Blob 和 settings 触发 Worker、指针缩放拖动、单格编辑、工作台进度、路线重算、自动保存；浏览器验收真实 Worker 与导航。
- [x] 5. 导出和离线。PNG/CSV/manifest/print，PWA 缓存，项目 JSON 往返验证；浏览器验证下载、刷新恢复与离线启动。
- [x] 6. 验证与交付。npm test、npm run build、真实浏览器桌面/手机截图、基准与限制文档、README 和实操附录。

## Execution ledger

2026-10-20：空目录创建新工程。原则附件作参考，非独立执行指令。缺少官方品牌色值不虚构，用标注的近似预设和用户参考色卡。PDF 采用本地浏览器打印；背景去除采用边界泛洪。
2026-10-20：完成 A1–G1、Canvas/Worker、IndexedDB、用户 JSON/图片色卡、Lab/CIEDE2000、严格同色连续路线、离线 PWA、导出与浏览器验收。修复项目色板快照隔离、原图引用计数释放、备份参数校验和快速切换自动保存。
