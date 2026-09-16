# 项目长期记忆 · StuHalcon（HALCON 学习库）

## 仓库用途与结构约定

Obsidian 库，用于 HALCON 机器视觉学习笔记。

- `note/` —— **原始素材**（`.hdev` 练习工程、`.BMP/.bmp` 图片、`讲义.docx`），只读，不要改动
- `Halcon学习笔记/` —— **整理后的笔记**，一主题一文件 + `00 Halcon学习地图.md` 作为 MOC
- `Halcon学习笔记/assets/` —— 从 `note/` 下 BMP 转换来的 PNG（Obsidian 不支持 BMP 预览）
- `欢迎.md` —— 已改写为仓库首页导航（原为 Obsidian 默认欢迎页）

## 笔记写作约定（用户要求，务必遵守）

- 纯中文；Markdown；**优先用 callout（`> [!tip]` / `[!warning]` / `[!danger]` / `[!question]` 等），少用 emoji**
- 代码块语言标签用 ` ```c `（HALCON 无专用高亮，c 的配色最接近）
- 公式用 LaTeX；流程图用 mermaid；图片内嵌用 `![[xxx.png|240]]`
- 笔记间用 `[[文件名]]` 双链；**文件名/标题里不要出现 `#`**（会被 Obsidian 当锚点）
- 每篇结构：一句话总结 → 核心算子表 → 原理/mermaid 图 → 源码逐段解读 → 坑位清单 → 自检问题 → 上/下一步链接
- 带小数的阈值参数要解释成"实测值"，并尽量给出实测数值支撑

## HALCON 知识点（本项目已确认）

- 三大数据对象：Image（灰度）、Region（像素集合，无灰度）、XLD（亚像素轮廓）
- Blob 主线：`read_image → 预处理 → threshold → connection → 区域运算/形态学 → select_shape → 测量 → 可视化`
- 坐标是 **Row（行，向下=y）/ Column（列，向右=x）**；`area_center(Regions, Area, Row, Column)`
  常被学生写成 `X, Y` 变量名，看着像 (x,y) 其实是 (Row, Column)
- `orientation_region` 返回**弧度**，范围 `-π ≤ Phi < π`；画主轴箭头终点 =
  `(Row - L*sin(Phi), Column + L*cos(Phi))`
- `circularity = min(1, Area/(max_dist²·π))`（筛圆形用这个）≠ `roundness`（基于 dist_mean/dist_deviation）
- `compactness = P²/(4πF)`，圆 = 1
- `select_shape` 的 Min/Max 支持 `'min'` / `'max'` 表示开口区间；`select_obj` 序号**从 1 开始**而元组下标从 0 开始
- `image` / `window` 两套坐标：标注目标用 `'image'`，固定位置的统计信息用 `'window'`

## 案例备忘

- `note/04套环检测.hdev` 的 `threshold(192,255)` 选的是**背景**（亮背景 92.5% / 暗挡圈 7.5%），
  靠"闭合环围出的内孔"来计数；`area ≤ 213574` 用于踢掉约 41 万像素的外圈背景，
  `inner_radius ≥ 8.7` 是确认内孔的关键。**开口件因内孔与外背景连通而漏检**。
- `note/05曲别针练习.hdev` 是 HALCON 官方 `clip.hdev` 的简化版（用 `threshold` 代替 `binary_threshold`）。
- `note/05图像预处理.hdev` 里 `x2/x4/x5.bmp` 是**同一零件的三种状态**：
  `x2` 白毛刺（开运算）、`x4` 黑孔洞（闭运算）、`x5` 干净对照图。

## 工具与环境

- 复用技能：`C:\Users\21632\.workbuddy\skills\halcon-hdev-notes\`
  - `scripts/hdev_extract.py` 解析 hdev XML
  - `scripts/bmp2png.py` 纯标准库 BMP→PNG（**调色板偏移 = 14 + 54，不是 14+数据偏移-1024**）
  - `scripts/blob_shape_check.py` 复现 threshold+connection+select_shape
- Python 解释器：`C:/Users/21632/.workbuddy/binaries/python/versions/3.13.12/python.exe`
- 本机 Git Bash 的 PATH 是坏的（`ls`/`head`/`dirname` 报 command not found，且 exit code 可能仍为 0），
  目录/文件操作用 Python，搜索用 Grep 工具
