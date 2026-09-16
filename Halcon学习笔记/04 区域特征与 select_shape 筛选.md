---
tags:
  - Halcon
  - 特征筛选
aliases:
  - select_shape
  - 区域特征
created: 2026-09-16
---

# 04 区域特征与 select_shape 筛选

> [!abstract] 一句话总结
> `select_shape` 是 HALCON 的"**筛子**"：对每个候选区域算出你指定的若干特征，**全部落在 [Min, Max] 区间内（`'and'`）或至少一个落在区间内（`'or'`）**的区域才被保留。

> **源文件：** `note/03二值化处理.hdev`、`note/04套环检测.hdev`
> **配套资料：** `note/讲义.docx` 中的特征清单（本笔记已完整整理并补充说明）

## 一、算子签名与参数

```c
select_shape (Regions, SelectedRegions, Features, Operation, Min, Max)
```

| 位置 | 参数 | 类型 | 说明 |
|---|---|---|---|
| 输入 | `Regions` | region-array | 候选区域（通常是 `connection` 的输出） |
| 输出 | `SelectedRegions` | region-array | 保留下来的区域 |
| 输入 | `Features` | string-tuple | 要检查的特征名，如 `['area','width']` |
| 输入 | `Operation` | string | `'and'`（全部满足）或 `'or'`（满足其一） |
| 输入 | `Min` | number-tuple | 每个特征的**下限**，与 `Features` 一一对应 |
| 输入 | `Max` | number-tuple | 每个特征的**上限**，与 `Features` 一一对应 |

**三条铁律：**

1. `Features`、`Min`、`Max` 三个元组**长度必须完全一致**，第 i 个特征对应第 i 个上下限。
2. 上下限是**闭区间**：`Min ≤ value ≤ Max` 才保留。
3. 边界可写字符串 `'min'` / `'max'` 表示"不设下限/不设上限"。

```c
* 单特征：只筛面积
select_shape (ConnectedRegions, SelectedRegions, 'area', 'and', 3500, 8107.97)

* 多特征 'and'：面积、宽度、高度 三个条件同时成立
select_shape (ConnectedRegions, SelectedRegions, \
              ['area','width','height'], 'and', \
              [527.98, 38.54, 54.38], [10454.1, 63.89, 1000])

* 开口区间：面积 >= 500，上不封顶
select_shape (ConnectedRegions, SelectedRegions, 'area', 'and', 500, 'max')
```

> [!tip] `'and'` 与 `'or'` 的语义
> - `'and'`：**所有**特征都要在各自区间内（**取交集，越筛越少**）—— 工业筛选 99% 用这个。
> - `'or'`：**只要有一个**特征在区间内就保留（**取并集，越筛越多**）—— 用于"大而圆 或 小而长"这类组合条件。

## 二、区域特征全表

下表按"用途"重新归类，比 HALCON 文档的字母序好记得多。**加粗**的是最常用的。

### 2.1 位置与尺寸（最常用）

| 特征名 | 含义 | 备注 |
|---|---|---|
| **`'area'`** | 区域面积（像素数） | 最核心的筛选依据，单位是像素 |
| **`'row'`** | 质心的行坐标 | 按位置筛选（如"只要上半区"） |
| **`'column'`** | 质心的列坐标 | 按位置筛选 |
| **`'width'`** | 外接矩形宽度（平行于坐标轴） | 即水平方向尺寸 |
| **`'height'`** | 外接矩形高度（平行于坐标轴） | 即垂直方向尺寸 |
| `'ratio'` | 高宽比 `height / width` | 判断瘦长物体 |
| `'row1'` | 外接矩形**左上角**行坐标 | 与 row2 配对使用 |
| `'column1'` | 外接矩形**左上角**列坐标 | |
| `'row2'` | 外接矩形**右下角**行坐标 | |
| `'column2'` | 外接矩形**右下角**列坐标 | |

> [!warning] 上下左右别搞混
> HALCON 文档写的是 **"upper left corner"（左上角）** 与 **"lower right corner"（右下角）**，因为 Row 向下增大。所以 `row1 < row2`、`column1 < column2` 恒成立。

### 2.2 形状相似度（判"像不像圆/椭圆/矩形"）

| 特征名 | 含义 | 计算与取值 |
|---|---|---|
| **`'circularity'`** | 圆度：与圆的相似程度 | $C = \min\left(1,\ \dfrac{F}{\max^2 \cdot \pi}\right)$，`F` 为面积，`max` 为**中心到轮廓像素的最大距离**。越接近 **1** 越像圆 |
| `'roundness'` | 圆度（轮廓均匀度） | 由轮廓到中心距离的**均值 `dist_mean`** 与**标准差 `dist_deviation`** 决定。越接近 **1** 越圆 |
| `'compactness'` | 紧密度 | $C = \dfrac{P^2}{4\pi F}$（`P` 为轮廓长）。圆 = **1**，越细长/越粗糙 → **越大** |
| **`'convexity'`** | 凸度 | 区域面积 / 凸包面积。完全凸 = 1，有凹陷 < 1 |
| **`'rectangularity'`** | 矩形度 | 区域面积 / 最小外接矩形面积。越接近 1 越"方" |
| `'anisometry'` | 等效椭圆的**轴比** | 长轴/短轴，越大越细长 |
| `'bulkiness'` | 松弛度（膨胀度） | 与 `anisometry`、`struct_factor` 同源（见 `eccentricity`） |
| `'struct_factor'` | 结构因子 | 同上 |
| `'num_sides'` | 多边形边数 | 用于区分三角形/四边形 |
| **`'contlength'`** | 轮廓长度 | 区域边界总长（单位像素） |
| `'dist_mean'` | 区域边界到中心的**平均距离** | `roundness` 的中间量 |
| `'dist_deviation'` | 区域边界到中心距离的**偏差** | 同上 |

> [!note] circularity 与 roundness 的区别（重要，容易混）
> 两者中文都常译作"圆度"，但**算法和用途完全不同**：
>
> | | `circularity` | `roundness` |
> |---|---|---|
> | 依据 | 面积 vs **最小外接圆**面积 | 轮廓到中心距离的**均值与标准差** |
> | 侧重点 | 整体"填满外接圆"的程度 | 轮廓的**均匀性** |
> | 判断圆形斑点 | **推荐用这个** | 正方形、窄边圆环也会接近 1，容易误判 |
> | 特例 | 窄边圆环 → 值很小 | 窄边圆环 → 值很大 |
>
> 结论：**做缺陷检测筛圆形斑点用 `'circularity'`**；`roundness`/`dist_mean`/`dist_deviation` 更适合分析轮廓均匀性。

### 2.3 椭圆与外接几何

| 特征名 | 含义 |
|---|---|
| `'ra'` | 等效椭圆的**长半轴** |
| `'rb'` | 等效椭圆的**短半轴** |
| `'phi'` | 等效椭圆的**方向**（弧度，来自 `elliptic_axis`） |
| **`'outer_radius'`** | **最小外接圆**的半径（`smallest_circle`） |
| **`'inner_radius'`** | **最大内切圆**的半径（`inner_circle`） |
| `'inner_width'` | 区域内**最大内接矩形**的宽度 |
| `'inner_height'` | 区域内**最大内接矩形**的高度 |
| `'rect2_phi'` | **最小外接矩形**的方向 |
| `'rect2_len1'` | 最小外接矩形的**半长**（长边的一半） |
| `'rect2_len2'` | 最小外接矩形的**半宽**（短边的一半） |
| **`'max_diameter'`** | 区域**最大直径**（轮廓上任意两点最远距离） |
| **`'orientation'`** | 区域的**方向**（弧度，来自 `orientation_region`） |

> [!tip] 外接 vs 内切，一句话分清
> - **`outer_radius` 是"把区域刚好框住"的最小圆半径 → 大**；
> - **`inner_radius` 是"刚好能塞进区域里"的最大圆半径 → 小**。
> - 同理 `width/height` 是**外接矩形**（框住区域），`inner_width/inner_height` 是**内接矩形**（塞进区域）。
>
> 判断"是不是有个洞/是不是空心"时，`inner_radius` 与 `outer_radius` 的**比值**非常有用。

### 2.4 拓扑结构（洞与连通性）

| 特征名 | 含义 |
|---|---|
| `'connect_num'` | **连通组件**的数目（一个区域被拆成几块） |
| `'holes_num'` | **孔的数目** |
| `'area_holes'` | 所有孔的面积之和 |
| `'euler_number'` | **欧拉数** = 连通组件数 − 孔数，用一个数字描述空间完整性 |

> [!note] 欧拉数的直觉
> 实心块：1 个连通块、0 个孔 → 欧拉数 1；甜甜圈：1 个连通块、1 个孔 → 欧拉数 0；两个圈：2 − 2 = 0。**欧拉数变化 = 拓扑结构变了**，适合检测"应为实心却出现穿孔"的缺陷。

### 2.5 几何矩（进阶）

| 特征名 | 含义 |
|---|---|
| `'moments_m11'` | 混合二阶矩，描述**非对称性**（对称区域 ≈ 0） |
| `'moments_m20'` | 沿 **Row** 方向的二阶矩（纵向伸展） |
| `'moments_m02'` | 沿 **Column** 方向的二阶矩（横向伸展） |
| `'moments_ia'` | 长轴方向的二阶矩（主轴惯性），`ia ≥ ib` |
| `'moments_ib'` | 短轴方向的二阶矩 |

**几何矩的意义（讲义补充）：**

- 零阶矩 $m_{00}$ → 反映目标**面积**
- 一阶矩 → 反映目标**质心位置**
- 二阶矩 → 又称**惯性矩**，反映伸展方向
- 三阶矩 → 表现对均值分布偏差的测度，即**扭曲度**
- 四阶矩 → 统计学中用于描述分布的**峰态**

`ia / ib` 比值常用来区分**裂纹（细长 → 比值大）**与**孔洞（圆形 → 比值接近 1）**。

## 三、配套筛选算子

| 算子 | 用途 | 示例 |
|---|---|---|
| `select_shape_std (Regions, Sel, ShapeFeature, Percent)` | 按"标准形状"筛选 | `'max_area'`（只留最大的）、`'original'`（与原区域相同） |
| `select_obj (Regions, Obj, Index)` | 按**序号**取一个区域 | `select_obj (Regions, Obj, 1)` 取第 1 个（**序号从 1 开始**） |
| `select_gray (Regions, Image, Sel, Features, Op, Min, Max)` | 按**灰度特征**筛选 | `'mean'`、`'max'`、`'deviation'` |
| `count_obj (Objects, Number)` | 统计**对象个数** | 等价于 `Number := \|Area\|`，但更快 |
| `sort_region (Regions, Sorted, SortMode, Order, RowOrCol)` | 按位置**排序** | `('first_point','true','column')` 从左到右排序 |

> [!warning] `select_obj` 与元组下标差 1
> `select_obj` 的序号**从 1 开始**，而元组下标 `Regions[0]` **从 0 开始**。这是最常见的越界/错位来源。

`sort_region` 的排序模式：`'character'`、`'first_point'`、`'last_point'`、`'upper_left'`、`'lower_left'`、`'upper_right'`、`'lower_right'`。

## 四、参数怎么定：从"测量"到"筛选"

新手最常问的是"上限定多少"。正确做法是**先测量再筛选**：

```c
* 第一步：先不筛选，把特征全部打出来看分布
area_center (ConnectedRegions, AllArea, Row, Col)
* 在 HDevelop 的变量窗口里查看 AllArea 的值域，或者显示到窗口上
for i := 0 to |AllArea| - 1 by 1
    dev_disp_text (AllArea[i], 'image', Row[i], Col[i], 'black', [], [])
endfor
stop ()
```

看到真实数值分布后，再取"**目标区间略宽、干扰区间之外**"的值填进 `Min`/`Max`。

> [!example] 源码中的参数是怎么来的
> **`03二值化处理.hdev`**（audi2 车标检测）：
> ```c
> select_shape (ConnectedRegions, SelectedRegions, \
>               ['area','width','height'], 'and', [527.98,38.54,54.38], [10454.1,63.89,1000])
> ```
> - 面积 `527.98 ~ 10454.1`：滤掉细小的噪点块，也限制最大块
> - 宽度 `38.54 ~ 63.89`：这是**很紧的窗口**，说明目标宽度相当固定 → 典型的"尺寸一致性筛选"
> - 高度 `54.38 ~ 1000`：下限卡高度、上限基本放开
> - 数值带两位小数，说明是**从实测数据里直接抄下来的**
>
> **`04套环检测.hdev`**（圆环计数）：
> ```c
> select_shape (ConnectedRegions, SelectedRegions, \
>               ['area','max_diameter','inner_radius'], 'and', [0,22.54,8.7], [213574,91.72,200])
> ```
> - 面积下限 `0` → **等于不限制**（保留所有非空区域）
> - `max_diameter 22.54 ~ 91.72` → 排除过小的碎块和过大的粘连块
> - `inner_radius 8.7 ~ 200` → **关键条件**：只有"中心有足够大的空洞"的区域才算圆环，这一步把实心杂质排除掉了

## 五、常见坑位清单

> [!danger] 三个必须记住的坑
> 1. **忘了 `connection`**：`'area'` 变成全体像素之和，筛选完全失效。
> 2. **`Min`/`Max` 与 `Features` 长度不一致**：报错或静默错位，特征和阈值张冠李戴。
> 3. **同一个区域内含多个物体**（没分开）：此时 `'area'` 是"合并面积"，会误保留。解决：`connection` 之后再筛，或用 `connection` + `union1` 组合。

其他注意点：

- 空区域不报错，但特征值为 0 / 空元组，可能在后续运算中产生意外结果。
- 筛选是**逐区域判断**，索引顺序保持不变，可以放心用"筛选前后的下标对应"。
- 筛选后若还要测量，用 `area_center (SelectedRegions, ...)`，**不要**再对 `ConnectedRegions` 测量。

---
上一步：[[03 二值化与连通域分割]] · 下一步：[[05 图像预处理与滤波]]
