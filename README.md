# StuHalcon

HALCON 机器视觉学习笔记库（Obsidian Vault）。

## 目录结构

```
StuHalcon/
├── Halcon学习笔记/           # 整理后的笔记（17 篇）
│   ├── 00 Halcon学习地图.md  # MOC：知识地图与索引，建议从这里开始
│   ├── 01 ~ 09 ...          # Blob 分析主线
│   ├── 10 算子速查表.md      # 按功能分类的算子索引
│   ├── 11 ~ 12 ...          # 仿射变换（矩阵 / 区域 / 轮廓 / 抠图）
│   ├── 13 ...               # 形态学调参专题（结构元半径标定）
│   ├── 14 ...               # 模板匹配（形状匹配 / 带缩放的 aniso 系列）
│   ├── 15 ...               # 2D 计量（卡尺测量 / 距离家族 / 先定位后测量）
│   ├── 16 ...               # 读码与 OCR（一维码 / 二维码 / 字符识别）
│   └── assets/              # 笔记内嵌图片（由 note/ 下的 BMP 转换而来）
└── note/                    # 原始素材（只读）
    ├── *.hdev               # HDevelop 练习工程（本质是 XML）
    ├── 仿射变换/*.hdev       # 仿射变换练习（01~07）
    ├── 模板匹配/*.hdev       # 模板匹配练习（01~03）+ 模版匹配/*.png 素材
    ├── 测量/*.hdev           # 2D 计量练习（01~05）+ my/ 子目录
    ├── 测量/测量/*.bmp        # 计量素材（0/2/3.bmp，同一零件不同姿态）
    ├── 测量/test*.bmp        # 点到线/点到点练习素材
    ├── 读码与OCR/*.hdev      # 读码练习（01~06）+ 字符识别（06字符识别）
    ├── 读码与OCR/End13/*.png  # EAN-13 素材（15 张）
    ├── 读码与OCR/qrcode/*.png # 工件上的 QR 码素材（9 张）
    ├── x1~x5.bmp            # 滤波与形态学练习素材
    ├── 形态学调整/1~5.bmp     # 形态学调参素材（与 x1~x5.bmp 同图异名，见笔记 13 附录）
    ├── 套环检测/*.BMP        # 套环检测素材（800×600）
    └── 讲义.docx            # select_shape 特征清单
```

## 笔记索引

| # | 笔记 | 内容 |
|---|---|---|
| 00 | [Halcon学习地图](Halcon学习笔记/00%20Halcon学习地图.md) | 三大数据对象、Blob 流程、算子命名规律、自检清单 |
| 01 | [语言基础与图像入门](Halcon学习笔记/01%20Halcon语言基础与图像入门.md) | 语法、元组、控制流、读图与窗口显示、Row/Column 坐标系 |
| 02 | [区域与区域集合运算](Halcon学习笔记/02%20区域与区域集合运算.md) | 交集、补集、反选、合并、对称差 |
| 03 | [二值化与连通域分割](Halcon学习笔记/03%20二值化与连通域分割.md) | `threshold` / `connection` / `union1` |
| 04 | [区域特征与 select_shape 筛选](Halcon学习笔记/04%20区域特征与%20select_shape%20筛选.md) | 特征全表、筛选写法与坑位 |
| 05 | [图像预处理与滤波](Halcon学习笔记/05%20图像预处理与滤波.md) | 灰度变换、均值/中值滤波、增强 |
| 06 | [形态学运算](Halcon学习笔记/06%20形态学运算.md) | 膨胀、腐蚀、开运算、闭运算 |
| 07 | [区域测量与结果可视化](Halcon学习笔记/07%20区域测量与结果可视化.md) | 面积、质心、角度、文字与箭头 |
| 08 | [实战 套环检测](Halcon学习笔记/08%20实战%20套环检测.md) | 遍历文件夹批量检测并计数 |
| 09 | [实战 曲别针计数与角度](Halcon学习笔记/09%20实战%20曲别针计数与角度.md) | 计数 + 方向主轴可视化 |
| 10 | [算子速查表](Halcon学习笔记/10%20算子速查表.md) | 按功能分类的算子索引 |
| 11 | [仿射变换矩阵与图像变换](Halcon学习笔记/11%20仿射变换矩阵与图像变换.md) | 齐次矩阵 `HomMat2D`、Row/Column 约定、`vector_angle_to_rigid`、反解参数 |
| 12 | [仿射变换实战 区域轮廓与抠图](Halcon学习笔记/12%20仿射变换实战%20区域轮廓与抠图.md) | 三对象变换、区域↔轮廓、`reduce_domain` / `crop_domain` 抠图 |
| 13 | [形态学调整 结构元半径标定](Halcon学习笔记/13%20形态学调整%20结构元半径标定.md) | 距离变换求半径下限、开/闭运算半径扫描标定、三个实测坑 |
| 14 | [模板匹配 形状匹配](Halcon学习笔记/14%20模板匹配%20形状匹配.md) | 抠 ROI 建形状模型、`create/find_shape_model` 参数全解、带缩放的 aniso 系列、仿射回显 |
| 15 | [测量模型 2D Metrology](Halcon学习笔记/15%20测量模型%202D%20Metrology.md) | 卡尺原理、`add_metrology_object_*_measure` 参数全解、`all_param` 下标映射、`distance_*` 距离家族、先定位后测量 |
| 16 | [读码与 OCR](Halcon学习笔记/16%20读码与%20OCR.md) | 一维码 / 二维码 / 文字 OCR 三条链路、码制清单、降噪排查顺序、`.occ`/`.omc` 分类器 |

## 学习路线

**主线（Blob 分析）**：`read_image` → 预处理（滤波/增强）→ `threshold` → `connection` → 区域运算/形态学 → `select_shape` → 测量 → 可视化

**支线（几何变换）**：测量拿到 `(Row, Column, Phi)` → `vector_angle_to_rigid` 构造 `HomMat2D` → `affine_trans_image` / `affine_trans_region` / `affine_trans_contour_xld` → `reduce_domain` + `crop_domain` 抠 ROI

**支线（模板匹配）**：`reduce_domain` 抠 ROI → `create_shape_model` 训练 → `find_shape_model` 查找 → `get_shape_model_contours` 取轮廓 → `hom_mat2d_*` + `affine_trans_contour_xld` 回显（目标大小会变时改用 `create_aniso_shape_model` / `find_aniso_shape_model`，见 [笔记 14](Halcon学习笔记/14%20模板匹配%20形状匹配.md)）

**专题（2D 计量）**：画近似形状 → `add_metrology_object_*_measure` 自动摆卡尺 → `apply_metrology_model` 拟合 → `get_metrology_object_result` 取参数 → `distance_pp` / `distance_pl` / `distance_cc` 算尺寸；零件会跑位时配合 `create_shape_model` + `reference_system` + `align_metrology_model` 做"先定位后测量"（见 [笔记 15](Halcon学习笔记/15%20测量模型%202D%20Metrology.md)）

**支线（识别）**：`create_bar_code_model` → `find_bar_code` → `get_bar_code_result`；或 `create_data_code_2d_model` → `find_data_code_2d`（难读时先 `median_image` / `gray_opening_shape` 降噪）；或 `create_text_model_reader` → `find_text` → `get_text_result`（见 [笔记 16](Halcon学习笔记/16%20读码与%20OCR.md)）

**专题（参数标定）**：`threshold` → 区分字符/缺陷 → `distance_transform` 求缺陷内切圆半径 → 贴着下限取 `opening_circle` / `closing_circle` 的 `Radius`（见 [笔记 13](Halcon学习笔记/13%20形态学调整%20结构元半径标定.md)）

配合 `note/` 下对应的 `.hdev` 练习，在 HDevelop 里按 `F6` 单步执行，逐段观察中间结果。

## 维护说明

- 笔记正文只改 `Halcon学习笔记/`，`note/` 作为原始素材保持不动。
- Obsidian 不支持 BMP 预览，笔记内嵌图片统一放在 `Halcon学习笔记/assets/` 下（PNG）。
- 笔记之间用 `[[双链]]` 关联；文件名与标题中不要出现 `#`（会被 Obsidian 当作锚点解析）。

## 自动同步

本库已安装 **obsidian-git** 插件（2.39.0），配置与 `StuC#` 库一致：

| 设置 | 值 |
|---|---|
| 自动提交 | 文件变更后触发，间隔 1 分钟 |
| 自动拉取 | 每 300 秒 |
| 自动推送 | 每 300 秒 |
| 启动时拉取 | 开启 |
| 同步方式 | merge（推送前先拉取） |

在 Obsidian 中打开本库后，插件即会按上述节奏自动 `commit → pull → push`。
手动同步可用命令面板（`Ctrl+P`）执行 **Git: Commit-and-sync**。

被 `.gitignore` 排除的本地文件（仅存在于本机，不随库同步）：
`.obsidian/workspace*.json`、`.obsidian/cache`、`.obsidian/plugins/obsidian-git/data.json`、
`欢迎.md`（本地首页导航）、`.workbuddy/`（含 `.workbuddy/memory/`）、`.trash/`。

