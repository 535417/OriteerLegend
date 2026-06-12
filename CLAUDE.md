# OriteerLegend — 项目上下文

## 项目概述
定向地图图例库，单页 HTML 应用。展示三套定向运动地图图例标准，支持浏览、搜索、详情查看。

**URL**: https://github.com/535417/OriteerLegend
**GitHub Pages**: https://535417.github.io/OriteerLegend/

## 文件结构
```
index.html          ← 唯一源文件，所有 HTML/CSS/JS/数据都在里面（约 120KB）
images/
  isom2017img/      ← ISOM 2017-2 图例 SVG（111 张）+ 旧 PNG
  issprom2019-2/    ← ISSprOM 2019-2 图例 SVG（100 张）+ 旧 PNG
  iscd2018/         ← ISCD 2018 图例 PNG（218 张）
```

## 架构要点

### 数据层（3 套图例）
- `ISOM_DATA` — ISOM 2017-2（112 条，SVG 图标，含 name_en 英文名）
- `ISSPR_DATA` — ISSprOM 2019-2（106 条，SVG 图标，含 name_en 英文名）
- `ISC_DATA` — ISCD 2018（172 条，PNG 图标，含 name_en 英文名）
- `ALL_DATA = [...ISSPR_DATA, ...ISOM_DATA, ...ISC_DATA]`
- ISOM/ISSprOM 数据格式：`{ code, name, name_en, description, icon, standard }`
- ISCD 数据格式：`{ id, name_zh, name_en, description, image, standard }`
- **注意**：ISCD 用 `id` 字段（如 "12.108"），ISOM/ISSprOM 用 `code` 字段（如 "101"）

### 路由系统（hash-based）
- 无 hash / `#` → "全部" 列表（ISSprOM 默认展开，ISOM/ISCD 折叠）
- `#tab/<std>` → 单标准分类列表
- `#detail/<std>/<code>` → 详情页

### 分类折叠
- ISOM/ISSprOM：按 100 地貌 / 200 岩壁 / 300 水系 / 400 植被 / 500 人工 / 600 技术 / 700 线路 分类
- ISCD：按列号分类 — 0(C列) / 1-5(D列) / 6(D列) / 8(E列) / 10-11(F列) / 12(G列) / 13(H列) / 15 强制通道
- 全部视图：三套标准各自可折叠；ISSprOM 默认展开
- 搜索视图：按标准分组，每个标准内按分类分组，默认全部展开
- **关键 Bug**：`getCategory()` 需用 `code || id` 兼容 ISCD 的 id 字段

### 三级返回（重要设计）
标签切换 push 历史 → 详情 push 历史 → 前后切换 replaceState。
返回键统一用 `history.back()`

### 搜索
- 仅匹配 `code/id + name`（不搜索 description）
- `toLowerCase()` 不区分英文大小写

### 英文名显示（name_en）
- ISOM/ISSprOM 的 name_en 从 SVG 文件名提取（如 `101-Contour.svg` → "Contour"）
- ISCD 已有 name_en
- 卡片列表和详情页：ISOM/ISSprOM/ISCD 都显示英文名小字（`.en-sub`）

### 描述换行渲染
- 描述中的 `\n` 在卡片列表通过 `.replace(/\n/g, '<br>')` 渲染
- 详情页：含 `\n` 的描述按行拆为独立 `<p>`；无 `\n` 的走原句号分段逻辑

### 关键函数签名
- `getCategory(code, standard)` — 返回分类 key，code 参数需兼容 id/code 双字段
- `renderList(items, groupByStd, isSearch)` — 三个模式：单标准分类 / 搜索结果 / 全部折叠
- `renderDetail()` — 渲染详情页，含描述分段逻辑
- `showStandard(std, skipHash)` — 显示列表
- `openDetail(std, code)` — 打开详情
- `searchAll(query)` — 全局搜索

## 编码约定
- 纯原生 JS + CSS，零依赖
- 变量命名：`$` 前缀 = DOM 元素引用
- 标准 key：`isom2017`, `issprom2019`, `iscd2018`
- 代码注释用中文
- 所有功能在一个 HTML 文件中
- 数据行极长（~30KB/行），用脚本批量修改，切勿手动

## 已知陷阱
- ISOM_DATA/ISSPR_DATA/ISC_DATA 都是单行巨型数组（约 30KB），手工编辑极易出错
- ISCD 字段名与其他两套不同（`id` vs `code`，`image` vs `icon`，`name_zh` vs `name`）
- 描述文本含未转义的双引号和中文标点，不可用 JSON.parse，只能用 eval
- SVG 图标尺寸不一（有的 mm 有的 px），且含大量留白，需 CSS 放大
