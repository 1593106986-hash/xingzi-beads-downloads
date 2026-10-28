# 星子拼豆：官网下载与多端安装方案

日期：2026-10-20。本文先于实现保存；用户提供的完整需求作为执行授权，常规选择由实现者决定。

## 产品边界与兼容性

应用统一命名为星子拼豆（Xingzi Beads），版本 1.1.0。保留 A1–G1 页面、50×50/100×100 转换、专属色板、CIEDE2000 感知匹配、路线算法、解释系统及低调的导航浮层。新增入口打开独立的关于/数据面板，不占用核心页面 ID，不截获原有导航快捷键。

不改 IndexedDB 名称 `bead-studio`、版本 1 或既有 `bead-*` 偏好键；安装在不同浏览器、域名和原生外壳的数据隔离，通过备份迁移。应用和官网均无账号、广告、遥测、上传接口和业务后端。首次安装提示非模态，可关闭并记忆。PWA、桌面、移动端共享前端和本地 Worker。

## 技术结构

- 应用：现有 React 18、TypeScript、Vite；PWA 改为 prompt 更新，静态 version.json 不进入预缓存，离线继续使用上次应用。更新由用户触发并先写入本地项目。
- 官网：独立 Vite 静态入口 `website/`，构建至 `dist-site/`。发布组合目录 `release-site/`：官网根目录、`app/` 下应用、`downloads/` 安装包。使用相对链接，兼容子路径、对象存储和内网。
- 桌面：Tauri 2，固定应用标识 `com.xingzi.beads`，默认 1280×860、最小 800×600；本地窗口状态插件、文件对话框、受限原生文件写入。默认无托盘/自定义菜单。Tauri 窗口使用 WebView 文件拖放，沿用导入页。Ctrl/Cmd+S 保存；Ctrl/Cmd+O 进入导入，不改原有工作台快捷键。
- 移动：Capacitor，Android/iOS 本地打包；相机和照片选择沿用显式触发权限的系统入口，移动文件导出通过本地 Filesystem + Share；Android 返回键先关闭新增面板，再回项目页，首页使用系统退出。保留既有双指缩放和平移，补全安全区和横屏适配。
- 数据：全部数据备份包含所有项目、项目色板、独立色板、原图和偏好；单项目备份复用该格式。AES-256-GCM + PBKDF2-SHA256（310000 次、随机 16 字节盐/12 字节 IV），密码可选且不存储。校验所有内容后以单个 IndexedDB 事务追加恢复，生成新 ID 避免覆盖现有项目；路线重新计算，保留制作进度、导航状态和设置。失败不改变原数据。
- 清空数据：明确二次确认，等待保存队列，清空项目/色板/图片/路线/进度和本应用偏好；重新载入，展示内置练习作品。

## 静态版本协议与更新

备份接口细化：`Backup = {format:"xingzi-beads-backup",schemaVersion:1,createdAt:string,projects:ExportManifest[],palettes:Palette[],preferences:Record<string,string>}`。每个项目单独携带其权威色板和可选原图 data URL。单项目模式仅导出对应项目/色板，全部模式还带未被项目引用的色板。恢复使用现有 `parseManifest`，为项目/色板/原图生成新 ID；完成格和导航游标保留，旧路线缓存舍弃。偏好只恢复白名单内值。输入大小上限 150MiB，所有内容校验后才写库。清空和恢复期间拒绝新的自动保存，等待旧队列再提交事务。

生命周期接口：`startDistribution(): () => void` 注册浏览器安装与可选检查监听并返回清理函数；`saveFile(blob:Blob,name:string): Promise<void>` 在 Tauri 使用保存对话框/本地写入、Capacitor 写缓存并调用系统分享、浏览器使用 Blob 下载，取消保存属于正常返回。相机/照片均由用户点击调起系统入口；浏览器不能保证同时显示相机和相册时提供两个入口。`useDistribution` 仅存储安装事件、待更新标记和面板开关，不访问项目算法。平台检测未知时推荐 PWA，始终允许手选平台；UA 无法可靠判断的架构必须在界面明确。

`version.json` 包含 schemaVersion、version、publishedAt、notes、downloads。每个下载条目包含 id、platform、arch、kind、file、size、sha256、minSystem、signed；平台含 windows/macos/linux/android/ios。不存在的安装包不出现在清单中；官网显示待发布和 PWA 入口，不能虚构大小或哈希。文件链接限制为本站 downloads/ 相对路径；商店链接单独配置并校验 HTTPS。

应用检查只请求编译时固定的静态 version.json（默认同源；原生构建可设置 VITE_VERSION_URL）；默认不自动请求，可手动或开启启动检查，离线静默。PWA 用 Service Worker 待更新提示，不主动 reload。原生移动只展示安装方式。Tauri updater 独立使用规范的静态 updater.json，构建时公钥与 HTTPS 固定源同时配置才开启，更新须用户点击；API 验签后安装，不能从普通版本清单取得任意可执行 URL。

## 官网内容与设计

首页、功能、下载、日志、系统要求、教程、隐私、常见问题、本地部署均为可直接锚点访问的静态内容。根据 UA 自动推荐平台，并允许手动选平台与 CPU 架构；macOS 架构不可靠自动猜测，默认明确标注 Apple Silicon，并展示 Intel。下载展示真实版本、发布日期、大小、SHA256、最低要求和签名状态。

设计属于 Extension：继承既有四颗拼豆图标 `public/icon.svg`，不重做核心 UI。官网以真实产品截图作主视觉。暖纸底 #f7f5f0、墨色 #29323b、陶土 #c7604c、鼠尾草 #839b80；中文使用本机苹方/微软雅黑，标题衬线仅官网；8px 间距，12–24px 圆角，单层柔和阴影，120–180ms 微动，尊重减少动态效果。视觉变化 4/10、动效 2/10、信息密度 5/10、素材依赖 6/10、品牌延续 9/10；手机按钮至少 44px，不用网络字体/CDN。

## 平台默认值和限制

- Windows 10 1809+ x64，依赖 WebView2（默认 bootstrapper 安装可能联网；离线分发见文档）。NSIS 安装包和免安装 exe；未签名触发 SmartScreen，发行者自备 Authenticode 证书。
- macOS 12+，Apple Silicon/Intel 分别构建 dmg/app；官网直下需 Developer ID 签名和 notarization，未签名包不可宣传为已验证。
- Linux 以 Ubuntu 22.04+、WebKitGTK 4.1 环境为基线；deb/AppImage，Wayland/发行版依赖需实机验证。
- Android 7+（实际 minSdk 取安装的 Capacitor 工程配置），APK 可直装；AAB 用于商店。发行者自备 keystore，安装器针对当前来源授权，建议安装后关闭。不请求全盘存储权限。
- iOS 最低版本以生成工程和 Capacitor SDK 为准；IPA 需 Apple 签名/描述文件。官网提供 App Store、TestFlight 和 PWA 方式；普通未签名 IPA 不能直接安装，不承诺企业分发对公众可用。
- PWA 需要 HTTPS 或 localhost；普通内网 HTTP 可浏览，但通常不能安装/离线缓存。Safari 手动分享→添加到主屏幕。浏览器可能清理存储，需导出备份。

## 构建与验证边界

后续维护为明确目标：品牌、版本、发布日期、静态官网/更新源集中于 `product.config.json`；官网文案独立模块，安装更新/备份/平台桥接相互隔离。提供双击启动、构建脚本、环境诊断与按修改目标索引的维护文档；保持原核心模块路径，方便继续优化。

提供本地命令、多平台 GitHub Actions、签名/公证模板和发布清单生成器。macOS/iOS 必须 macOS/Xcode，Android 需要 JDK/SDK，桌面需要 Rust 与系统构建工具。当前环境缺少这些工具时先完成可运行 Web/PWA、静态官网与原生工程配置，不能声称已构建或已安装原生包。

验收覆盖：原有全部单元测试、备份加密/错误密码/篡改/合并恢复、版本清单拒绝不安全链接、关闭更新不发请求、官网桌面/手机布局、真实下载校验、PWA 离线、更新等待确认和原有解释/路线导航回归。100×100 转换和路线性能记录真实测试结果；60fps 与相机、签名、安装需目标设备验收。
