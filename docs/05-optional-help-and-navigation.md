# 可选解释与非阻塞步骤导航：更新方案

## 默认选择与原有规则

沿用 React18/TS/Zustand/Canvas2D/Worker/idb/PWA，不新增运行时依赖，不改颜色映射、严格同色面积与路线排序。默认标准模式、解释开启、胶囊紧凑、底边吸附、透明度 88%、中速快进、震动关闭。默认不自动打开抽屉；新手提示仅是可跳过的页内一行。

单击前进/后退、长按后退、+5/+10、同色/色块/分区跳转只移动导航光标。长按前进会把经过的豆子标记为完成，松手后定位下一颗未完成豆；整段为一个可撤销操作。滑条向前实时标记经过的步骤，回滑撤回连续滑动新增的完成标记，保留滑动前已完成的豆子；松手后再次拖动、刷新后均可延续这段滑动的回退依据，受最近80条导航历史限制，手动操作会建立新基线。“跳过色块”保留未完成格，并记录断点；“回到断点”或“下一未完成”可找回。完成计数始终来自唯一格子集合，当前步骤即使已完成也可回看。

## 更新页面与组件

永久 A1–G1 保留。应用顶栏增加常驻“术语与偏好”入口；B1/C1/D1/E1/F1/G1 加按需解释标记和新手页内提示。F1 增加画布下方边缘导航栏及区域选择工具，原当前颜色、计时和清单保留。

```
App
 ├─ HelpLauncher / PageGuide（轻量索引，无整本词典）
 ├─ A1 … G1 / Term、HelpTrigger
 └─ lazy HelpPanel（非模态，无 backdrop、无焦点陷阱）
     ├─ 搜索 + 分类 + 阅读偏好
     └─ 一句话 / 可展开详情 / 可展开 SVG 示意与例子
F1 Workbench
 ├─ BoardCanvas（平移、缩放、点按、长按、框选/圈选）
 ├─ SelectionTools（一次提交区域完成/撤销）
 ├─ NavigationDock（边缘停靠，不覆盖 Canvas）
 │   ├─ CompactCapsule / DragHandle / RepeatButton
 │   ├─ StepScrubber（预览直到松手才提交）
 │   └─ JumpActions / BatchActions / NavigationPreferences
 └─ CurrentColor / Timer / Progress / RemainingList
```

桌面解释是右侧小卡，手机是底部抽屉，最高 65vh；仅卡片自身接收指针事件，背景 Canvas 和控制条继续可用。可拖标题下滑关闭、Escape 关闭、不自动夺焦。桌面悬停显示一句话，点问号显示详情；长按名词 550ms 打开，移动超过 8px 取消。全关解释后禁用这些内容入口，顶栏偏好入口仍可恢复。

导航栏保留在 Canvas 外的专用边缘区域，不以 fixed 浮层覆盖豆格。拖手柄可吸附底/顶/左/右（手机左右按可用宽度降为顶/底），下滑折叠；3秒无操作只降低透明度。展开需主动点按。暂停隐藏当前/下一高亮并缩成胶囊；恢复保留位置。专家可隐藏导航栏，用工作台“显示导航”恢复，手势/快捷键仍可用。

## 数据结构与状态

词典 `src/features/explain/dictionary.json` 与 UI 分离：

```ts
type ExplanationMode = 'beginner' | 'standard' | 'expert';
interface ExplanationEntry {
 id:string; name:string; category:'图像处理'|'颜色科学'|'路线算法'|'应用技术'|'操作功能';
 aliases:string[]; summary:string; details:string[]; relation:string;
 example:string; diagram?:'colors'|'dither'|'route'|'grid';
 effects?:{on:string;off:string;recommended:string;impact:string};
 related:string[]; target?:{page:PageId;anchor?:string};
}
interface HelpPreferences {
 enabled:boolean; mode:ExplanationMode; read:string[];
 dismissedPages:string[]; expanded:Record<string,{details:boolean;example:boolean}>;
}
interface NavigationCommand {
 added:number[]; removed:number[]; before:number|null; after:number|null;
 label:string; beforePaused:boolean; afterPaused:boolean;
 beforeBreakpoints:number[]; afterBreakpoints:number[];
}
interface NavigationState {
 cursorCell:number|null; paused:boolean; breakpoints:number[];
 past:NavigationCommand[]; future:NavigationCommand[];
}
```

帮助与导航外观偏好分别使用 Zustand store 写入带 `version:1` 标记的本地 localStorage；组件只订阅需要字段。轻量词条索引在主包；详细词典 JSON 与抽屉作为动态 chunk，构建后由 VitePWA 预缓存，断网首次点帮助可用。详情与示意的折叠状态、已读提示、跳过页、模式持久化；不开云请求。

项目新增可选 navigation 字段，旧项目自动默认。导航与进度一起写原 IndexedDB projects/progress，schema v1 向后兼容；完整备份包含导航与最近80条增量历史。done 仍是一组唯一有效 cellIndex，批量操作只存 added/removed 差量，一次区域操作或一次长按完成为一次撤销。导航改变不增加 project.revision、不清除 route、不请求 Worker。网格/色板/图像/路线参数变化会重建路线并清理旧导航增量历史，防止跨不同图纸撤销；名称/锁定等不改变路线的变更保留已计算路线并修订缓存版本。历史恢复前检验路线指纹、命令游标链与格子边界；浏览跳转只更新 cursorCell，长按前进则提交经过格子的完成差量。

## 流程、快捷键与防误触

1. 新手点 ? → 一句话 → 按需展开“详细说明”和“示例”。卡片支持复制、相关术语、定位对应页面设置、“我知道了”。复制失败可选择文字，无云端回退。
2. 导航单击只浏览 ±1；长按前进按住350ms后按慢/中/快400/180/80ms逐颗标记完成，松手时一次提交一条可撤销命令并跳到下一未完成豆。长按后退只浏览。移出、指针取消、失焦、切页均停止；默认不震动。
3. 滑动实时更新目标颜色、坐标、Canvas完成淡化、进度与剩余清单；从连续滑动的起点之后到目标步骤之间的豆子加入预览完成集，回滑从基线重新派生。松手/键盘确认一次性提交 added/removed 与光标，不逐帧写 IndexedDB 或创建历史；每次提交均可整段撤销/重做。Esc、pointercancel、丢失捕获或窗口失焦丢弃本次预览。NavigationCommand.scrubOrigin 记录滑动起点；连续滑动通过逆向还原最近的滑动命令求出原始完成基线，非滑动命令会分隔一段滑动。+5/+10/下一同色/下一块/下一分区仍只浏览，不改变 done。
4. 点有效格只增加完成（反复点不反转），自动移到下一未完成；豆格长按只撤销该格。步骤按钮长按前进会完成经过的豆，松手自动移到下一未完成；长按后退仅浏览。
5. 选“框选”后拖矩形，“圈选”后描多边形；平移模式仍保留。显式“完成选区/撤销选区完成”后一次提交，不边拖边写。双指始终缩放并取消区域笔划，避免误选。
6. 完成当前块剩余项一条命令；跳过块记录断点但不完成；没有其他块时只提示，不把光标置空；断点最多20个。暂停同样可撤销。
7. F1 内空格下一步、Shift+空格上一步；左右箭头 ±1，上下箭头 ±10，Ctrl/Cmd+Z 撤销、Shift+Ctrl/Cmd+Z 或 Ctrl/Cmd+Y 重做。输入框、下拉框、按钮、可编辑区域和解释面板内不接管快捷键。

## 上下文准确性与性能

推荐颜色说明会显示当前格色号、色板与选色限制；手动改过的颜色明确说明是“当前指定颜色”，不伪称实时最优。转换平均色差高于20时只出现内联提醒，可点解释，不自动弹窗。路线说明显示实际模式、总同色数量、当前连通块面积与工作区；2-opt/TSP/WASM 明确为术语与扩展方向，当前实现未启用，不伪造能力。

每次路线改变时 O(N) 建立 cell→step、颜色/块/分区索引；跳转采用预建索引或有界扫描，不做重算。区域命中 O(N) 上限10000，只在松手运行；Canvas圈选最多保留512个抽稀后的轨迹点，绘制选框/圈线不更新 React 格子列表。批量更新 O(done+selection)，历史存差量且最多80条；Canvas 静态豆层不因导航重建。按组件/模式定位到同一项目下的路线过期后校验项目和 revision，取消中的长按不提交旧光标。

## 验证

单测覆盖词典完整性、相关ID/类别、无公式门槛与示例、跳转不完成、重复完成幂等、批量/断点/撤销重做、历史游标链校验、备份往返以及10k格增量预算。真实浏览器检查离线词典首次打开、三种模式记忆、抽屉开启时Canvas与步骤区都可操作、长按快进停止与单次撤销、滑动预览、框选与圈选批量、鼠标四边停靠/键盘、刷新断点与历史、移动下滑/拖动/双指缩放、无外部请求；保留原全量回归。
