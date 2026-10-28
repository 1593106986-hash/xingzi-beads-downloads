# 豆序 Bead Studio：完整产品与技术方案

## 1. 定位、默认值与边界

面向在手机、平板与电脑上制作拼豆的用户，完成「导入 → 校色 → 规划 → 逐豆制作 → 导出」。数据仅留在设备，首次安装依赖可用网络下载 npm 包，应用运行没有云端、账号、广告、远程字体、分析脚本或图像 API。PWA 首次访问后可离线使用，部署需 localhost 或 HTTPS。

默认：50×50、照片模式、最多 24 色、关闭抖动、保留透明空格、四邻接连通域、蛇形块内顺序、严格同色连续、不分区。默认“大色面优先”按每种颜色的总豆数降序，整色拼完后换色；颜色内部按连通块面积降序。100×100 可选 25×25 或 50×50 工作区。空格不是透明色豆，不进入路线与用量。

用户提供的 principles.txt 是参考材料，其中合理的组织理念用于模块化、页面永久编号和扩展接口。附件色卡为未校准 JPEG，无法认定品牌与官方 RGB；220 个取样的 A/B/C/D/E/F/G/H/M 系列保留色号，称为「附件参考色卡」，H1 透明不作为实体颜色。另提供原创与通用基础预设，不冒充品牌官方数据。用户可通过 JSON 导入任意品牌的精确色值/色号，也可上传自己的色卡图片、图纸提取专属色板，提取后支持校色、改号、改名、增删及导出。

本地背景去除采用边界颜色泛洪，不宣称语义抠图。图纸识别采用周期网格检测、格内中心取样和线条避让，复杂透视、遮挡、反光、文字需手动裁剪与校对；不内置 OCR，不虚构识别色号。PDF 使用浏览器打印生成，免额外 PDF 依赖并支持中文。

## 2. 页面与用户路径

| 永久 ID | 页面 | 主要内容 |
|---|---|---|
| A1 | 项目收藏 | 项目缩略图、示例、导入、新建、打开、重命名、删除、恢复离线项目 |
| B1 | 导入与裁剪 | 文件/相机、照片/图纸、正方形裁剪位置与缩放、90°旋转、确认 |
| C1 | 像素编辑 | 大画布、50/100 网格、图像调节、色数、抖动、单格替色、锁定、矩形选择、局部重算、四种视图 |
| D1 | 色板管理 | 参考预设、图片提色、图纸专属色板、JSON 导入导出、增删改颜色、色号名称与感知色差 |
| E1 | 路线规划 | 三种模式、分区、蛇形/最近邻/边界优先、路线统计与重算 |
| F1 | 拼豆工作台 | 当前颜色、坐标、当前/下一格、完成淡化、点按完成、长按撤销、计时、撤销重做 |
| G1 | 统计与导出 | 总量/剩余/分区清单、色号 PNG、分色 PNG、进度 PNG、CSV、路线 JSON、项目 JSON、打印 PDF |

桌面采用窄侧栏 + 主工作区 + 参数侧栏，移动端采用顶栏、可缩放画布、可展开的设置面板和底部导航。默认展示可直接编辑的本地原创示例，避免空白起步。项目画布编辑与工作台共用渲染组件与手势层，页面、功能和核心引擎各自独立目录。

## 3. 架构与扩展接口

固定 React 18 + TypeScript + Vite + Zustand + Canvas2D + Web Worker + idb + CSS 变量 + Vitest + vite-plugin-pwa。

```
src/app                         应用壳与页面路由
src/pages/A1-projects           项目列表
src/pages/B1-import             导入/裁剪
src/pages/C1-editor             像素编辑
src/pages/D1-palettes           色板管理
src/pages/E1-route              路线设置
src/pages/F1-workbench          拼豆工作台
src/pages/G1-export             统计导出
src/features/board              Canvas 与指针手势
src/features/import            解码与裁剪参数
src/features/export            PNG/CSV/JSON/打印
src/core/color                 sRGB/Lab/CIEDE2000
src/core/image                 网格映射/提色/网格检测
src/core/route                 连通域与规划
src/workers                    类型化通信与任务调度
src/storage                    IndexedDB v1
src/store                      项目状态、撤销重做、持久化
src/data                       离线示例与色板
src/shared                     类型与通用 UI
tests                          数学、图像、路线、存储、浏览器验收
```

替换接口：ImageEngine、RouteEngine、ProjectRepository、ExportAdapter。算法只依赖领域类型，不依赖 React；Worker 统一接收 map/extract/route 任务。local-map 子区域参数属于 map 协议，锁定格始终不被覆盖。主线程只处理文件读取、状态、绘图与下载；图像解码/缩放、像素处理、颜色匹配、提色、图纸检测、连通域与路线均在 Worker 内执行。任务 ID + 最新结果门禁防止旧响应覆盖新参数；请求队列合并预览，传输 ImageBitmap 或 Blob，无主线程计算回退。

## 4. 数据模型与存储

详见 src/shared/types.ts。核心模型：

* PaletteColor：id、code、name、hex、可选 lab；code 在同一色板唯一。
* Palette：id、name、brand、source、colors、updatedAt、referenceOnly。
* GridCell：index、colorId/null、locked；坐标由 index % size 和 floor(index/size) 得到。
* Project：id、name、size(50|100)、paletteId、paletteSnapshot、cells、settings、sourceAssetId、mappingKey、routeOptions、done、elapsedSeconds、revision、createdAt、updatedAt。paletteSnapshot 隔离共享色卡修改；mappingKey 防止重新进入页面覆盖手工编辑。
* RouteStep：order、cellIndex、colorId、componentId、zone。
* RoutePlan：projectId、revision、mode、steps、components、distance、colorChanges、durationMs、options。
* WorkerRequest/Response：带 id、type 的可判别联合，成功数据或可读错误。
* ExportManifest：schemaVersion、project、palette、route、progress、generatedAt、source(原图 data URL、名称与 MIME)。恢复时新建身份、校验范围与色板闭包，缓存路线重新计算。

IndexedDB 数据库 bead-studio，version=1：projects(key=id, index updatedAt)、palettes(key=id)、routes(key=projectId)、progress(key=projectId)、assets(key=id, Blob)。项目/色卡/路线/进度与首次导入图片使用单个读写事务保存，写操作顺序排队。500ms 防抖自动保存，切换项目时保存离开的对象，visibilitychange 时即时刷新。删除最后一个引用项目后释放原图。完成状态按格子索引存，重新规划仍保留进度；网格尺寸变化清空进度并可撤销。历史最多 40 次快照，持久化仅保存最终状态。存储失败明确提示并保留内存状态，提供完整项目 JSON 备份。

## 5. 颜色映射算法

以 D65 白点 (95.047,100,108.883) 将标准 sRGB 线性化后转 XYZ，再转 Lab。匹配采用标准 CIEDE2000，kL=kC=kH=1，处理低色度与 hue 跨 360°。公式通过 Sharma 参考测试对校验。

```text
sRGBToLab(rgb):
  c = rgb / 255
  linear(c) = c/12.92 if c<=0.04045 else ((c+0.055)/1.055)^2.4
  XYZ = D65_sRGB_matrix * linear(rgb) * 100
  f(t) = cbrt(t) if t>(6/29)^3 else t/(3*(6/29)^2)+4/29
  [fx,fy,fz] = f(X/95.047), f(Y/100), f(Z/108.883)
  return [116*fy-16, 500*(fx-fy), 200*(fy-fz)]

CIEDE2000(lab1, lab2):
  compute chroma C1,C2 and G adjustment from mean chroma
  adjust a -> a'; derive C', hue h' in [0,360)
  compute ΔL', ΔC', ΔH'; wrap hue difference to [-180,180]
  compute mean lightness/chroma/hue (zero chroma is a special case)
  derive SL,SC,SH and hue rotation RT (Sharma 2005)
  return sqrt((ΔL'/SL)^2 + (ΔC'/SC)^2 + (ΔH'/SH)^2
              + RT*(ΔC'/SC)*(ΔH'/SH))

nearest(sample, palette):
  if exact 24-bit RGB cache hit: return cached result
  for entry in precomputed Lab palette:
    if |ΔL|/SL >= currentBestDelta: skip  // exact lower bound
    evaluate full CIEDE2000; retain minimum
  cache and return a real PaletteColor ID
```

单次颜色转换/色差为 O(1)，单次匹配最坏 O(P)，预计算 O(P)，所有匹配输出都通过当前色板 ID 引用。下界剪枝与全表穷举的相等性有随机样本测试。

```text
map(image, palette, options, previousCells?, region?):
  decode and crop/rotate image with worker OffscreenCanvas
  if diagram: estimate periodic grid lines, align sampling rectangle
  resample to working resolution; alpha-aware mean samples for photo
  or use inner-cell samples avoiding grid borders for diagram
  if background removal: flood boundary-similar pixels -> alpha=0
  apply brightness/contrast/saturation/edge enhancement
  labPalette = sRGBToLab(each palette color)
  for each nonempty sample:
    nearest = argmin CIEDE2000(sampleLab, labPalette)
    histogram[nearest]++
  candidates = top K by mapped frequency, K<=palette.size
  for row in rows, x in columns:
    if locked or outside region: retain previous cell
    if alpha < threshold: empty
    else:
      q = nearest perceptual match(sample + propagated error, candidates)
      output.colorId = q.id
      if dithering: Floyd-Steinberg distribute RGB error (7,3,5,1)/16
  validate every nonempty cell belongs to palette
```

抖动使用当前色板子集重新映射后才写入格子，误差不会作为输出色。缓存完全相同的 RGB → 匹配项，不用近似量化缓存破坏精确匹配。透明样本不参与平均，避免黑边。色数限制对重新映射区域生效；已锁定颜色计入预算，锁定本身超过上限时保留锁定并提示。

设格子数 N<=10000、色板色数 P、色数上限 K：颜色匹配 O(NP+NK)，空间 O(N+P)。局部重算仍在 Worker 完整采样并匹配 N 格，只写回区域 R，因此匹配复杂度仍为 O(NP+NK)、写回 O(R)。照片先在 Worker 缩至 800×800，再取格均值。UI 的“转换耗时”包含完整 Worker 图像任务的解码、旋转、裁剪、图纸检测与映射，不含队列等候和主线程绘制。桌面耗时不作为所有手机性能保证。

专属色板：低分辨率图像颜色直方图 → 去除透明/近黑细线及白边候选 → 感知距离聚类 → 频率排序 → 返回最多 64 个中心颜色，并生成 U01 等色号。色卡图有文字时取平坦色块，附件预设直接使用色块左侧无字区域的稳健中值。

## 6. 路线算法与覆盖不变量

```text
components = floodFill4(cells, exactly same palette color ID)
groups = group components by exact palette color ID
for color: totalArea[color] = sum(component.area)
colors = sort by totalArea descending, then stable color ID
while colors remain:
  color = first(colors)
  blocks = groups[color] sorted by area descending, zone, component ID
  while blocks remain:
    block = first(blocks)
    if few-colors: choose nearest entry among equal-largest blocks (up to 64)
    if short-distance: choose nearest entry among next 64 blocks + zone penalty
    indices = snake | boundary-first | nearest-neighbor(block)
    append every index with its actual zone to route; remove block
  remove color  // global continuity: finish ALL its zones before next color

coverage = bitmap(N)
for step in route:
  assert cell not empty AND !coverage[cell]
  coverage[cell]=true
assert route.length == count(nonempty) AND every nonempty visited
```

三种模式的颜色顺序都固定为同色总面积降序，整色连续完成。默认与“少换色”在颜色内部大块优先；少换色在相同块面积时选最近入口。“少移动”在当前颜色内最多 64 个候选块中按入口距离选择，可改变同色内部面积顺序。非空颜色数为 P 时，换色数固定为 P−1（空图为 0），已经达到严格同色约束下的最小值，统一换色惩罚不会进一步改变下界。分区不切开真实四邻接连通块，避免跨区大块的面积被低估；其作用是块内蛇形排序、块间距离评分与每步工作区标记。

连通域与覆盖校验 O(N)，蛇形排序总计 O(N log N)，组件排序 O(C log C)，颜色排序 O(P log P)。数组移除/候选扫描最坏 O(C² + P²)，候选评分上限 64 块。块内最近邻在块大于 512 格时自动用蛇形，空间 O(N+C+P)。当前使用最近邻近似，无全局 TSP 最优保证；2-opt 留作可替换算法。结果显示实际换色数、欧氏格距与 Worker 耗时。

## 7. 交互与视觉规范

品牌「豆序」，手工工作桌气质：冷白纸色 #F5F6F8、画布 #FFFFFF、墨色 #20262E、次级灰 #78808B、主强调陶粉 #C7604C、拼豆图真实色作为唯一丰富颜色来源。暗色底 #181C22、面板 #222830、文字 #EDF0F4。亮暗模式是全局 tokens，不反转图像。

中文系统字体 PingFang SC / Microsoft YaHei，英文系统 sans-serif，数字 tabular-nums。标题 28/36、页面副标题 18/26、正文 14/22、元信息 12/18；间距 4/8/12/16/24/32，面板圆角 16、控件 10、豆格圆角随缩放。桌面侧栏 220px、参数栏 288px，移动断点 760px，触控热区至少 44px。细边框与轻阴影仅用于浮层，核心是带坐标标尺的拼豆画布。

Canvas 根据 DPR 绘制，静态格层离屏缓存，当前/下一位置用覆盖层，避免每秒重绘全部。手势通过 Pointer Events 合并 requestAnimationFrame，双指缩放、单指拖动、点按、500ms 长按。色号在可读尺寸显示；缩小时隐藏密集文字。完成状态透明度 0.25，下一步脉冲只作用于指示框。支持 prefers-reduced-motion、焦点环、明确按钮文本/aria-label、键盘撤销 Ctrl/Cmd+Z。

## 8. 导出、部署与验收

PNG：色号图、分色图、进度图；CSV：含色号、名称、总数、完成、剩余和区域；JSON：完整项目 manifest 与路线步骤。打印页嵌入本地生成的 PNG、色号清单与路线概要，用户选择「另存为 PDF」。CSV 防公式注入，JSON 导入验证尺寸、色号、格索引、唯一 id、进度与路线，禁止外部资源依赖。

开发 npm install / npm run dev；生产 npm run build / npm run preview -- --host 0.0.0.0。dist 可部署任意静态服务、支持相对 base 路径。PWA precache 打包 JS/CSS/图标/示例，无外部 CDN。相机与 PWA 安装在手机需要 HTTPS；局域网 HTTP 可用文件导入，无法承诺相机/离线缓存权限。Capacitor/Tauri 可封装 dist，但本交付为可运行 Web/PWA，不预装原生 SDK。

测试覆盖：CIEDE2000 参考值；50/100 网格、透明、抖动和调色后色板闭包；色数上限、局部/锁定；三种路线全覆盖/不重复、大块降序、分区；空图、单色、棋盘格最坏输入；IndexedDB 事务与恢复；历史撤销；桌面/手机视口、实际 Worker、PNG/打印、离线刷新、暗色与持久化。实测写入 docs/03-verification.md，已知限制写入 README。
