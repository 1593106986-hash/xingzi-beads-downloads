# 分发验收清单

## 当前主机可直接验证

- [x] `npm run doctor`：Node v24.17.0；Rust/Cargo、Java、Android SDK 与 Xcode 未安装，原生包需对应平台环境。
- [x] `npm test`：12 个测试文件、59 项测试通过。
- [x] `npm run build:release`：应用、PWA 与官网构建通过并生成 `release-site/`；当前没有真实原生包，downloads 为空。
- [x] `npm run test:distribution`：桌面/手机布局、平台切换、完整 SHA256 展示、维护文档可达、无外部请求通过。
- [x] `npm run test:release-script`：真实文件 SHA256 生成与路径穿越拒绝通过。
- [x] `npm run test:e2e`：发布版真实图片映射、路线、色板、导出、备份恢复、离线重载和移动端缩放通过。
- [x] `npm run test:features`：离线解释、路线快进/回退、滑动与批量标记、撤销、快捷键和移动端工作台布局通过。
- [x] `npm audit`：0 vulnerabilities。
- [ ] 在界面手动检查加密备份、错误密码、篡改文件与清空确认；自动化单元测试已覆盖加密和恢复校验。

## 目标平台设备验收

- [ ] Windows 10 1809+ x64：安装、卸载前恢复、文件拖拽、Ctrl/Cmd+S/O、窗口尺寸记忆、Authenticode 签名/SmartScreen。
- [ ] macOS 12+ Apple Silicon/Intel：DMG、签名公证、相机/文件导入、原生 updater。
- [ ] Ubuntu 22.04+：AppImage/deb、WebKitGTK、Wayland/X11、保存对话框。
- [ ] Android 7+：APK/AAB、相机/相册权限、返回键、旋转与安全区、系统分享、同签名覆盖更新。
- [ ] iOS 15+：相机/照片用途说明、横竖屏、返回/导航、App Store/TestFlight Archive、PWA 添加到主屏幕。

## 性能口径

在目标设备冷启动、无开发者工具、单次 50×50 与 100×100 图片转换分别记录 Worker `durationMs`；路线记录 planner `durationMs`。100×100 转换目标约 1 秒、路线约 2 秒是待目标设备验证的指标，不能用开发机测试结果替代。浏览器 DevTools Performance 和桌面 60fps 采样用于发现交互掉帧；不把“代码检查通过”写成性能保证。

## 隐私回归

- [ ] 页面请求只包含本地静态资源和用户主动点击的静态 version.json；无遥测、图片上传、账号或第三方 CDN。
- [ ] 官网下载只返回静态包，不触发安装和原生权限。
- [ ] Android 云备份/设备迁移明确排除应用数据库；iOS 隐私清单声明不收集数据。
- [ ] 原生 updater endpoint 只接受 HTTPS、固定公钥和 Tauri `.sig`；普通 version.json 不能安装可执行文件。
