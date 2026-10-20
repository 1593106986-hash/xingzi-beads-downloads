# 夜间模式全链路主题审计

## 审计清单

严重程度：P0 为首屏或核心工作区割裂，P1 为高频页面/浮层差异，P2 为外围流程或明确的非主题白色内容。

| 组件 / 页面 | 严重程度 | 审计发现与修复 |
| --- | --- | --- |
| 应用根节点、路由首屏、加载态 | P0 | 首屏前未解析偏好会造成浅色底闪；在 HTML 内联解析 `bead-theme` 及系统偏好，根节点预置 `data-theme` 和 `color-scheme`，加载态使用主题变量。 |
| 编辑、路线、工作台 Canvas | P0 | 画布底色、空格、网格、完成态、路线/选区覆盖层若仍使用亮色常量会形成大片漏白；Canvas 从 CSS 读取语义令牌并在主题事件中重绘，进度和路线状态保存在原 store，不随主题重建。 |
| 项目、导入、编辑、色板、路线、工作台、导出页面 | P1 | 页面共用变量化表面、文字、边框、弱表面；真实拼豆色值保留原样，它们是作品数据，不是 UI 主题。 |
| 底部工作控制条、路线高亮、解释卡片 | P1 | 半透明表面和阴影改为主题适配，暗色下提高遮罩密度、减少发白的材质高光。 |
| 导航抽屉、解释面板、分发设置、Modal、Toast、Tooltip、菜单 | P1 | 浮层使用 `surface-elevated`、`scrim`、主题阴影和边框；自定义 Tooltip 从 `data-tooltip` 读取内容。 |
| 日期/颜色/文件选择相关控件 | P1 | 浏览器原生弹层由根级 `color-scheme` 跟随主题；页面控件、下拉项、焦点态和滚动条使用主题令牌。 颜色选择器的色样仍展示准确拼豆颜色。 |
| 图片、缩略图、上传占位、裁剪区 | P1 | 图片承载容器/透明 PNG 棋盘格采用暗色棋盘令牌；图片像素本身不做反色。 |
| 骨架屏、空状态、错误/成功态 | P2 | 使用 `skeleton`、`surface-soft`、语义状态色；不再出现浅色占位卡片。 |
| 第三方组件及系统表单控件 | P1 | 项目没有统一第三方 UI 组件库；Phosphor 图标继承文本色，控件通过主题适配层统一。 `color-scheme: dark` 启用原生暗控件，range/checkbox/select 和滚动条自定义。 |
| PWA 浏览器外壳 | P1 | 首屏脚本同步 `theme-color` 并选择单一亮/暗 manifest；主题切换更新 meta 与 manifest，包含 `background_color`。 |
| Capacitor 系统栏及原生启动屏 | P2 | Capacitor 状态栏跟随运行时主题；Android/iOS 原生启动屏在 WebView/JavaScript 启动之前无法读取 Web `localStorage`，因此只可使用静态启动资源，无法保证首次启动画面按用户偏好变化。 |
| localStorage / IndexedDB 旧偏好 | P2 | `bead-theme` 仅接受 `system`、`light`、`dark`，缺失/无效值迁移为 `system`。 主题不写入项目 IndexedDB，旧项目/路线进度不需要迁移。 |

## 硬编码颜色边界

业务 UI 的中性色、状态色、表面色应引用语义令牌。审计时发现的白色/固定颜色用法主要有以下明确例外：

- `src/features/export/export.ts` 的白纸底、网格与黑色标注属于可下载/打印图纸的输出规范，不能随应用界面暗色改变。
- `src/data/palettes.ts`、用户自定义色板以及各页面的 `color.hex` 是真实拼豆色值，必须保持物理色卡准确；白豆仍应显示白色。
- `src/core/image/mapping.ts` 中白色填充是图像透明度/去背景算法的输入值，不是组件样式。
- 令牌文件内出现白色只用于浅色主题、深色模式下珠子高光，或数据图表上的浅/深色文字标签。不要把这些数据/输出颜色误删成主题变量。

其余 `src/app/styles.css`、导航、解释及分发样式中的 UI 背景/边框/文字/阴影统一引用令牌。 `white-space` 等包含 white 字样的 CSS 属性与颜色无关。

## 语义令牌表

CSS 变量位于 `src/app/theme-tokens.css`。下表是核心令牌；颜色值为 sRGB 十六进制或原样列出的透明色。

| 令牌 | Light | Dark |
| --- | --- | --- |
| `background` | `#f5f6f8` | `#151a20` |
| `surface` | `#ffffff` | `#1d242c` |
| `surface-elevated` | `#ffffff` | `#252e38` |
| `surface-soft` | `#f9fafb` | `#222b34` |
| `overlay` | `rgba(255,255,255,.96)` | `rgba(37,46,56,.97)` |
| `border` | `#75818d` | `#6c7d8e` |
| `divider` | `#eceef1` | `#2d3742` |
| `text-primary` | `#29323b` | `#edf2f6` |
| `text-secondary` | `#68737f` | `#b2bdc8` |
| `text-disabled` | `#9aa4af` | `#74808b` |
| `accent` / `accent-hover` | `#ad4938` / `#983e30` | `#e9947f` / `#f2aa96` |
| `accent-soft` | `#fcf0ec` | `#3b2b2b` |
| `success` / `warning` / `danger` | `#3d754f` / `#8c5c17` / `#b14e42` | `#9ed1aa` / `#e2b46b` / `#ee8b8b` |
| `shadow` | `0 12px 35px rgba(36,44,59,.12)` | `0 16px 40px rgba(0,0,0,.42)` |
| `scrim` | `rgba(18,24,32,.48)` | `rgba(6,10,15,.66)` |
| `canvas-bg` / `cell-empty` | `#f7f8fa` | `#202830` |
| `grid-line` / `grid-line-major` | `rgba(93,105,118,.2/.3)` | `rgba(171,188,204,.22/.34)` |
| `cell-selected` | `rgba(65,141,175,.28)` | `rgba(90,176,211,.3)` |
| `cell-done` | `rgba(246,248,250,.78)` | `rgba(13,20,27,.6)` |
| `cell-next` / `highlight` | `rgba(199,96,76,.2)` | `rgba(239,155,134,.22)` |
| `skeleton` | `#e9edf1` | `#2b353f` |
| `checker-a` / `checker-b` | `#e6e9ed` / `#f7f8fa` | `#2a333d` / `#202830` |

附属令牌还覆盖强调色文字、选择描边/遮罩、完成态底色、珠子高光/阴影、焦点环、数据标签和空格弱化态，见 `theme-tokens.css`。暗色背景采用分层中性灰而非纯黑；主要及次要文字分别为浅灰 `#edf2f6` / `#b2bdc8`。关键文字对比度可按 WCAG 2.2 AA 4.5:1（普通文字）和 3:1（大字/控件边界）验收；焦点使用高亮强调色令牌。

## 状态、持久化和平台行为

- 偏好为三态：`system`（默认）、`light`、`dark`；系统偏好变化只在 `system` 下生效。首屏脚本与 React 复用同一 `bead-theme` 键；旧的 `dark`/`light` 字符串兼容，非法值归一化到 `system`。
- IndexedDB 只保存作品、色板与制作数据，不保存主题。主题切换不修改项目对象、不重建路线、不清空完成进度。
- Canvas 使用 `readCanvasTheme()` 获取已解析的 CSS 色值并监听 `bead-theme-change` 重绘；CSS transition 只应用于小型控件/表面，根背景和画布不做大面积动画。
- 控件暗色由根 `color-scheme`、表单样式和滚动条规则组合实现；透明图片棋盘格用 `checker-a`、`checker-b`。
- PWA 根据解析主题选择 `manifest-light.webmanifest` 或 `manifest-dark.webmanifest`。浏览器主题变化时会更新 `theme-color`、Apple 状态栏样式和 manifest 链接。
- Capacitor 运行时使用对应的 StatusBar 样式。Android/iOS 的原生冷启动图展示发生在 Web 偏好读取前，不能使用浏览器存储做按偏好切换；网页首屏随后会依照保存偏好渲染。
- 项目没有第三方 UI 组件库；所用图标集只输出 SVG 图标，颜色通过继承主题文字色控制。

## 改动文件

- `src/app/theme-tokens.css`、`src/app/styles.css`：唯一 CSS 主题令牌及全局、组件样式暗色覆盖。
- `src/app/theme.ts`、`src/store/appStore.ts`、`src/app/App.tsx`：首屏主题应用、偏好状态/系统监听与切换。
- `src/features/board/BoardCanvas.tsx`：画布颜色读取、切换重绘和深色透明棋盘。
- `src/features/distribution/PlatformEffects.tsx`：Capacitor 状态栏主题同步。
- `src/features/explain/HelpPanel.tsx`、`src/shared/TooltipHost.tsx`：解释面板与 Tooltip 主题适配。
- `src/pages/A1-projects/ProjectsPage.tsx`：项目页插画颜色令牌化。
- `vite.config.ts`、`index.html`、`scripts/theme-assets.mjs`、`scripts/theme-assets.d.mts`：首屏主题资源与按主题生成 PWA manifest。
- `tests/theme-audit-browser.mjs`：浏览器端主题回归检查。

## 验证方式

```powershell
npm test
npx tsc --noEmit --pretty false
npm run build
npm run build:site
npm run dev -- --host 127.0.0.1
$env:TEST_URL = 'http://127.0.0.1:5173/'
node tests/theme-audit-browser.mjs
```

浏览器检查涵盖暗色首屏解析、亮暗切换及 manifest 单例、原生控件颜色方案、Canvas 首像素、切换主题时制作进度保留和页面脚本错误。原生启动图主题动态适配受上述 WebView 启动时序限制。
