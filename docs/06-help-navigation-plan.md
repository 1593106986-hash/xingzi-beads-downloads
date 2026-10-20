# Help and Navigation Implementation Plan

**Goal:** 在现有 A1–G1 增加完全本地的可选解释与可撤销步骤导航。
**Architecture:** 词典/索引/UI分离；导航使用项目内增量命令，外观用独立偏好store；Canvas新增显式区域选择。
**Tech Stack:** 沿用现有固定栈，无新运行时依赖。
**Spec:** docs/05-optional-help-and-navigation.md
**Execution:** 当前会话实现，用户已要求可运行代码；使用既有合理默认值授权，不额外停在设计确认。

## Global Constraints

不改核心映射和严格同色路线；默认不自动弹窗；所有解释本地；普通快速跳转不完成豆子、不增加图纸 revision，长按前进将经过的豆标记完成并保持原子撤销；偏好/断点/增量历史离线持久化。

## Review Focus

长按中取消/离开控件；输入框快捷键；跨重算历史；旧项目导入；打开帮助后Canvas与导航仍可操作。

## Tasks

- [x] 1. `shared/navigation.ts` / `features/navigation/model.ts` + tests/navigation.test.ts：差量命令、纯跳转、批量、断点、备份往返、游标链校验、10k格操作。
- [x] 2. `features/explain`：词典、轻量索引、持久化模式/偏好、术语长按、新手提示、lazy 非模态抽屉与搜索详情；校验唯一性、必需词、类别及相关词链。
- [x] 3. `features/navigation`：快进/后退原子撤销、跳转、滑动、显式矩形/圈选、增量历史；F1快捷键和画布分区接入。
- [x] 4. 页面解释入口、上下文/实际颜色与块面积、提醒、桌面/手机体验测试；51单测、原 browser 回归、附加离线/持久化全流程。
- [x] 5. 两轮只读审查，修复跨页进度撤销分叉、路线扫描性能、已完成末尾、移动抽屉占位和双指取消基线；更新说明与新增验收自动化。
- [x] 6. 最终全量测试、build、刷新服务、截图与交付：11个测试文件/51项单测、基础浏览器回归、新功能离线与移动端浏览器流程均通过；生产构建通过，预览服务可访问。
