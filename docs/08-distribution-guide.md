# 星子拼豆分发与维护指南

## 本机发布 Android 与 Windows（2026-10-20）

公开网站：<https://1593106986-hash.github.io/xingzi-beads-downloads/>。网站、网页版与安装包部署在同一静态目录；发布仓库为 `1593106986-hash/xingzi-beads-downloads`。

```powershell
npm run android:build
npm run desktop:build -- --bundles nsis
node scripts/collect-native.mjs windows x64
```

Android 命令生成正式签名的 APK 和 AAB，位于 `artifacts/packages/`；APK 可直接安装，AAB 用于应用商店。构建默认只使用已备份的发布密钥；只有明确传入 `-CreateNewKey` 才会生成新密钥，避免误建新钥导致老用户无法覆盖安装。密钥和受 Windows DPAPI 保护的 credential 文件存于 `%LOCALAPPDATA%\XingziBeads\signing`，不会上传到网站或仓库。务必保留此目录；credential 文件只能由原 Windows 用户/机器解密。换机器前运行 `备份签名密钥.cmd`（或 `scripts/backup-signing.ps1`），把密钥、明文 `密码.txt` 和说明备份到安全位置，并确保备份目录不公开。不要把签名目录作为普通网站备份发布。

收集 Windows 后，把 Android APK 条目合并到 `artifacts/release-input.json` 的 `downloads` 数组，设 `platform: "android"`、`arch: "universal"`、`kind: "apk"`、`signed: true`，`source` 指向真实 APK。Windows 的 NSIS 安装器与便携版目前没有 Authenticode 证书，必须保持 `signed: false`，首次运行可能出现 SmartScreen 提示。NSIS 会按需安装 WebView2 Runtime。

日常更新官网直接双击 `更新官网.cmd`（桌面快捷方式 `3-更新官网.lnk`）。菜单有三项：

1. 只重新生成：版本号不变，适合改样式或文案。
2. 发布新版本：先改版本号和更新说明，再生成。
3. 只上线现有 `release-site/`：不重新生成。

生成完成后会询问是否直接上线，默认不上线。确认上线后会调用 `scripts/publish-github.ps1 -SkipBuild` 上传，并等待 GitHub Pages 生效、核对线上版本和主资源；Pages 构建失败时会自动重试一次。下载区安装包版本与当前版本不一致时会警告并默认取消，再次确认后才只更新网页/PWA。需要当前账号对发布仓库的写权限以及 `gh` 登录或 `GH_TOKEN`。

也可以手动执行：

```powershell
npm run build:release
npm run deploy -- -SkipBuild
```

部署命令先核对所有真实包的 SHA256，再上传 `release-site/` 并启用 GitHub Pages；它不会上传源码目录或签名材料。发布后应等待 Pages 构建完成，并下载线上 APK/EXE 核对 `SHA256SUMS.txt`。未来版本需保留同一 Android 签名密钥及应用标识 `com.xingzi.beads`，同时递增版本号。

网络阻止 Git HTTPS 推送时，部署命令会回退到已认证的 GitHub API；也可直接传入 `-UseApi`。公网文件校验可运行 `node scripts/verify-online.mjs`，通过 HTTPS Range 分段下载、重组并核对所有文件的完整 SHA256。可选参数为网站 URL 和指定解析 IP，TLS 仍校验真实域名。

## 先跑起来

Windows 双击 `启动星子拼豆.cmd`，或执行：

```bash
npm ci
npm run dev:all
```

应用：`http://localhost:5173`；官网：`http://localhost:5174`。普通开发只运行 `npm run dev`。环境检查：`npm run doctor`。

## 静态官网发布

```bash
npm run check
npm run build:release
npm run preview:release
```

把 `release-site/` 的全部内容上传到任意静态服务器、对象存储或内网服务器。应用放在 `/app/`，官网根目录；服务器必须正确返回 `application/javascript`、`application/manifest+json`、`application/json` 和图片 MIME。使用 HTTPS，PWA 才能完整安装和离线运行；`localhost` 仅适合开发。

`product.config.json` 是产品信息唯一来源。每次发版先修改版本、日期和日志，运行 `npm run prepare:distribution`。没有真实包时 `version.json` 的 downloads 必须为空，官网只提供 PWA。生成实际安装包后，准备 `artifacts/release-input.json`（可复制 `artifacts/release-input.example.json`），执行：

```bash
node scripts/release-manifest.mjs artifacts/release-input.json release-site
```

脚本复制真实包、计算 SHA256，生成 `version.json`、`SHA256SUMS.txt` 和 `CHANGELOG.md`。不接受跨域、路径穿越、JavaScript 协议、重复文件名、空文件或未签名 iOS IPA。将清单和下载包放在同一静态发布目录。

## 桌面端

安装 Rust、Cargo、系统构建工具和 Node 22+。Windows 还需 WebView2 Runtime；Linux 需要 WebKitGTK 4.1 开发包；macOS 需要 Xcode Command Line Tools。执行：

```bash
npm run desktop:dev
npm run desktop:build
```

默认生成 Windows NSIS/便携文件、macOS DMG/app、Linux AppImage/deb。构建脚本会拒绝缺失 Rust 的环境。窗口状态插件保存尺寸和位置；文件权限只通过用户选择的保存对话框授予。应用不含托盘和业务后台。

代码签名：`TAURI_SIGNING_PRIVATE_KEY` 是 Tauri updater 更新包签名密钥，不等同于安装包的操作系统代码签名。Windows 的 Authenticode 证书需由发布者另行配置；macOS 用 Developer ID 签名并由 `notarytool` 公证；Linux 包按发行版签名策略发布。当前 CI 模板不会为 Windows/Linux 安装包声称已完成代码签名。Tauri updater 只有同时配置 `TAURI_UPDATER_PUBLIC_KEY`、`TAURI_UPDATER_ENDPOINT`、`TAURI_SIGNING_PRIVATE_KEY` 时才启用；endpoint 必须 HTTPS。`node scripts/generate-updater.mjs` 只能从真实 `.sig` 生成静态 updater JSON。

## Android 与 iOS

Capacitor 原生工程已在 `android/` 与 `ios/`。Android 当前基线 minSdk 24、target/compile SDK 36，需要 Android Studio、SDK、JDK/其内置 JDK：

```bash
npm run build
npx cap sync android
npx cap open android
```

在 Android Studio 里用自己的 keystore 生成 signed APK/AAB；CI 读取 `ANDROID_KEYSTORE_FILE`、`ANDROID_KEYSTORE_PASSWORD`、`ANDROID_KEY_ALIAS`、`ANDROID_KEY_PASSWORD`，仓库不保存 keystore。直接 APK 安装需用户允许当前来源，商店发布优先 AAB。

iOS 基线为 iOS 15，需要 macOS、Xcode、Apple Developer Team：

```bash
npm run build
npx cap sync ios
npx cap open ios
```

在 Xcode 选择 Team、Bundle Identifier `com.xingzi.beads`，运行真机或 Archive。App Store Connect/TestFlight 使用签名 Archive；官网不提供普通未签名 IPA。`ios/ExportOptions.example.plist` 只是模板，不能直接发布。相机/照片用途说明和 `PrivacyInfo.xcprivacy` 已加入工程。

## 数据、升级与回滚

原有 IndexedDB 名称 `bead-studio` 和 `bead-*` 偏好键不可改。更新前在应用内导出全部数据；升级不会主动清库，换浏览器/域名或清除网站数据后用 `.xingzi` 备份恢复。加密备份使用本地 AES-256-GCM，密码不会保存。

PWA 更新进入等待状态，用户在“安装 / 关于 / 数据”确认后先保存项目再刷新；版本检查默认关闭，只读取同源 `version.json`，离线静默。桌面 updater 只信任 Tauri 签名和 HTTPS endpoint；移动端跟随商店/原生安装包规则。出现坏版本时暂停该版本清单或回退静态目录，保留上一版下载包和哈希。
