# 实操附录：复用、调试与扩展

## 想改什么，改哪里

| 目标 | 文件 | 说明 |
|---|---|---|
| 改应用名、版本、日期、更新日志 | `product.config.json` | 唯一产品配置；改完运行 `npm run prepare:distribution` |
| 改官网页面、文案或下载布局 | `website/src/site.tsx` | 官网独立于应用；样式在 `website/src/site.css` |
| 改官网颜色、间距或响应式 | `website/src/site.css` | 维护 `--paper/--ink/--accent/--sage`，手机断点为 600px |
| 改应用安装/关于/备份面板 | `src/features/distribution/DistributionPanel.tsx` | 备份规则在 `backup.ts`，加密在 `crypto.ts` |
| 改平台桥接（桌面保存、手机相机） | `src/features/distribution/platform.ts` | 浏览器降级为 Blob 下载和 file input |
| 改 PWA 安装/更新行为 | `src/features/distribution/runtime.ts` | 不要在这里强制 reload 或上传项目 |
| 改桌面权限和窗口 | `src-tauri/tauri.conf.json`、`src-tauri/capabilities/default.json` | 原生工程使用 `npm run desktop:build` |
| 改 Android/iOS 显示名和版本 | `scripts/prepare-native.mjs`、`capacitor.config.ts` | 运行 `npm run build && npx cap sync` 后再打开 IDE |
| 改发布下载包、大小、SHA256 | `artifacts/release-input.json` | 只写真实文件，运行 `npm run release:manifest` |
| 修改核心转换/色差/路线 | 保持原 `src/core`、`src/workers` 入口 | 不把本地算法搬到官网或原生层 |

最少维护流程：`npm run doctor` → 修改 `product.config.json` 或目标模块 → `npm run check`。本地预览用 `npm run dev:all`（应用 5173、官网 5174）；发布静态目录用 `npm run build:release`。真实安装包生成后，先运行 `node scripts/collect-native.mjs ...` 再生成校验和，不手工填写 `version.json`。

## 永久页面与责任边界

A1 项目、B1 导入、C1 编辑、D1 色卡、E1 路线、F1 工作台、G1 导出各有固定目录。修改界面时先到 `src/pages` 对应目录；修改领域规则时进入 `src/core`，通过 Worker 协议交付结果。页面编号不因标题变化而更换。

| 修改目标 | 入口 | 验证 |
|---|---|---|
| 接入新品牌色卡 | D1 的 JSON 导入或 `src/data/palettes.ts` | 色号唯一、颜色有效、来源真实 |
| 改匹配算法 | `src/core/color/color.ts` | Sharma 标准样本、穷举等价测试 |
| 改图纸识别 | `src/core/image/diagram.ts` | 双轴周期检测、图片/图纸浏览器导入 |
| 改路线策略 | `src/core/route/planner.ts` | 覆盖、同色连续、总面积排序、跨区连通面积 |
| 更换持久化 | `ProjectRepository` / `src/storage/db.ts` | 原子保存、保存顺序、共享原图删除 |
| 增加导出形式 | `ExportAdapter` / `src/features/export` | 断网下载、中文打印、JSON 往返 |
| 调整视觉 | `src/app/styles.css` 的 CSS 变量 | 390px、桌面、暗色、减少动态效果 |

## Worker 请求流程

1. 主线程只提供 Blob、参数、色板和原格子，不在 UI 中遍历颜色做重匹配。
2. `engine.request` 分配请求 ID，Worker 返回带 ID 的类型化结果；失败须完成 Promise，不能留下永远等待的忙碌状态。
3. 连续调整参数时，Worker 合并尚未执行的预览；主线程以项目 ID、revision 和任务 token 再检查一次，旧结果不能覆盖新项目。
4. 保存 `mappingKey`：包含源图 ID、色板 ID/色值、网格尺寸和图像参数。页面再次进入时签名一致就保留手工改色，不自动重画。新建未转换项目显式设空键；旧项目缺键时按已保存网格兼容，恢复备份则无条件生成当前签名。撤销图像参数也能恢复原结果。自动映射 effect 订阅 revision，避免转换期间编辑后旧回包被拒绝却不重试。
5. 手动重算是单独历史记录；自动预览属于参数修改的同一记录。锁定格与区域外格子始终保留。

## 数据保护

IndexedDB v1 包含 projects、palettes、routes、progress、assets。数据库连接复用，写操作按调用顺序排队，项目、色卡快照、路线、进度和首次导入原图在同一事务中提交。切换项目先保存离开的对象，不能依靠页面卸载后的防抖。

项目保存独立 `paletteSnapshot`。其他项目修改共享色板后，重新打开旧项目会恢复为单独色板；所有格子写入都校验颜色归属。项目备份恢复时生成新的项目/色板身份，保留内部颜色 ID，不采信外部 Lab 缓存和路线缓存。

删除项目时仅释放没有其他项目引用的原图。清除浏览器网站数据仍会删除所有作品；“统计与导出 → 备份完整项目”可下载包含原图的 JSON。历史保留当前会话最近 40 次，跨刷新恢复保存结果而不恢复撤销栈。

## 本地部署与测试

```bash
npm ci
npm test
npm run build
npm run preview -- --host 0.0.0.0
```

另一个终端运行 `npx playwright install chromium`（仅测试工具初装下载），然后 `npm run test:e2e`。可用 `PLAYWRIGHT_CHROMIUM_EXECUTABLE` 指向已安装的 Chromium。默认测试 URL 为 `http://localhost:4173`，用 `TEST_URL` 覆盖。截图与下载产物位于 `test-results`。

将 `dist` 放在任意支持静态文件的本地 HTTP/HTTPS 服务；不要直接双击 index.html 使用 file 协议。手机局域网访问开发服务可用电脑的局域网 IP，安装 PWA、相机权限和离线功能应使用 HTTPS。部署新版本后由 Service Worker 更新缓存；数据保存在当前站点来源，换域名/端口视作另一个存储空间。

Capacitor/Tauri 可把 `dist` 作为 Web 前端资源，但须另配原生 SDK、权限与应用签名；本项目不包含原生安装包。新增语义抠图、OCR 或 WASM 时应打包本地模型并置于 Worker，不接云服务。
