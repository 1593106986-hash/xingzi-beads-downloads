# 分发验收记录

验证日期：2026-10-20；版本：1.1.0。本记录更新了此前“缺少原生工具链、尚无安装包”的状态。

## 构建与文件

- Windows：Rust/MSVC release 编译与 NSIS 打包成功，安装器 2,633,277 字节；另提供 11,552,256 字节的便携 EXE。
- Android：JDK 21、Gradle 8.14.3、SDK 36 构建 `assembleRelease bundleRelease` 成功。直接安装 APK 7,062,264 字节；AAB 留在本机供商店发布。
- APK：`apksigner verify --verbose --print-certs` 通过，APK Signature Scheme v2，RSA 3072 位。应用标识 `com.xingzi.beads`，versionCode 10100，最低 Android 7（API 24），目标 API 36，包含 arm64-v8a、armeabi-v7a、x86、x86_64。
- 签名证书 SHA256：`7f6549b5c0fd1ef2cedb3fdba11382ae61e84d3959becd0b085c262bb6d4450c`。签名材料仅存本机私有目录，网站无私钥或密码。
- Windows Authenticode 状态为 `NotSigned`，发布清单如实标注 `signed: false`。未配置静默原生自动更新；用户下载安装新版。

## 已通过的验证

- `npm test`：13 个测试文件、63 项测试通过。
- `npm run test:release-script`：真实文件 SHA256、路径穿越拒绝通过。
- 应用、PWA、官网构建通过，发布清单包括 Windows 安装器、Windows 便携版、Android APK 三个真实文件。
- `npm run test:e2e`：真实图片映射、Worker、路线、色板、编辑与进度持久化、PNG/CSV/PDF、备份恢复、离线重载、暗色模式和移动布局通过。
- `npm run test:features`：离线解释、手势、快捷键、撤销、批量标记和移动工作台通过。
- `npm run test:distribution`：官网桌面/移动布局、平台切换、完整 SHA256 展示与文档访问通过。
- `npm run test:packages`：浏览器实际点击 Windows 和 Android 下载，核对下载大小及 SHA256；另核对便携 EXE，检查网页版入口可用，页面无 JavaScript 错误。此项使用真实发布清单，不模拟安装包。
- Windows NSIS 实际安装到隔离验证目录，启动安装后的原生程序，WebView 正常渲染；原生 API 返回版本 1.1.0。随后执行静默卸载。

安装器内 EXE 与便携 EXE 有三个字节的差异，这是 Tauri 为 NSIS 设置安装包类型标记的结果；两者分别按各自分发文件的 SHA256 验证。

## 下载校验值

| 文件 | SHA256 |
| --- | --- |
| Windows 安装器 | `138f4b7e9530ddec40a9262af8e74e3523023b3f3b1414ac6a8f5e4ea06bb66b` |
| Windows 便携版 | `d9383ef67fa47db1cbacc6b15a276cb7e861d7b0701109c007d22c19e6f76681` |
| Android APK | `78d59ca50f1680f3c39a6860c77d70a42ba3db504dc56b060e0241110031f5c0` |

公开网站：<https://1593106986-hash.github.io/xingzi-beads-downloads/>。线上验证使用 `TEST_URL` 指向此地址运行 `npm run test:packages`；需要检查 Pages 构建成功和线上实际下载的校验值，不能仅以“上传成功”代替。

## 验证边界

本机未连接 Android 真机，也没有已配置的模拟器，因此尚未进行手机安装、相机/相册授权、系统分享、返回键和覆盖更新的实机验收。Windows 已验证当前电脑上的安装、启动、界面和卸载；最低版本系统、全新无 WebView2 环境仍需独立设备验收。Windows 首次下载可能出现 SmartScreen 提示。

macOS、Linux、iOS 不在本次安装包交付范围。性能指标需要在具体目标设备上测量，不能从浏览器测试推导手机或最低配置设备的速度。
