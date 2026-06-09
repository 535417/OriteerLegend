# OriteerLegend — 项目上下文

## 项目概述
定向地图图例库，单页 HTML 应用。展示三套定向运动地图图例标准，支持浏览、搜索、详
情查看。

**URL**: https://github.com/535417/OriteerLegend
**GitHub Pages**: https://535417.github.io/OriteerLegend/

## 文件结构
```
index.html          ← 唯一源文件，所有 HTML/CSS/JS/数据都在里面（约 150KB）
images/
  isom2017img/      ← ISOM 2017-2 图例 PNG（213 张）
  issprom2019-2/    ← ISSprOM 2019-2 图例 PNG（约 120 张）
  iscd2018/         ← ISCD 2018 图例 PNG（218 张，从 base64 解码）
ISCD2018.htm        ← 原始 ISCD 2018 报告（OpenOrienteering Mapper 生成）
```

## 架构要点

### 数据层（3 套图例）
- `ISOM_DATA` — 国际定向地图规范 2017-2（213 条，详细中文描述）
- `ISSPR_DATA` — 国际短距离定向地图规范 2019-2（约 120 条）
- `ISC_DATA` — 国际检查点说明规范 2018（218 条，描述较简要）
- `ALL_DATA = [...ISOM_DATA, ...ISSPR_DATA, ...ISC_DATA]`
- `DATA_BY_STD = { isom2017:, issprom2019:, iscd2018: }`
- 每条数据格式：`{ code, name, description, icon, standard }`

### 路由系统（hash-based）
- 无 hash / `#` → "全部" 列表
- `#tab/<std>` → 分类列表（如 `#tab/isom2017`）
- `#detail/<std>/<code>` → 详情页

### 三级返回（重要设计）
标签切换 push 历史 → 详情 push 历史 → 前后切换 replaceState。
返回键统一用 `history.back()`，实现：详情 → 分类列表 → 全部 → 退出。

### 搜索
- 仅匹配 `code + name`（不搜索 description）
- `toLowerCase()` 不区分英文大小写

### 关键函数名
- `showStandard(std, skipHash)` — 显示列表，skipHash 跳过历史记录
- `openDetail(std, code)` — 打开详情
- `navigateDetail(direction)` — 前后切换（+1/-1）
- `handleRoute()` — hash 路由处理器
- `showListPage()` — 纯 DOM 切换（不操作历史）
- `searchAll(query)` — 全局搜索
- `renderList(items, groupByStd)` — 渲染卡片列表
- `renderDetail()` — 渲染详情页

## 最近完成的修改

1. **三级返回** — hash 路由支持 `#tab/...`，详情→分类→全部→退出
2. **713/714 描述清理** — 删除末尾多余的勘误表内容
3. **ISCD 2018 集成** — 从 ISCD2018.htm 自动解析 218 个符号
4. **搜索优化** — 仅搜编号+名称，ISCD 加入索引

## 编码约定
- 纯原生 JS + CSS，零依赖
- 变量命名：`$` 前缀 = DOM 元素引用
- 标准 key：`isom2017`, `issprom2019`, `iscd2018`
- 代码注释用中文
- 所有功能在一个 HTML 文件中
