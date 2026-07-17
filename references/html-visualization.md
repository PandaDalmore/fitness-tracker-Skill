# HTML 可视化模板使用指南

## 概述

Skill 提供三个 HTML 模板：两个可视化报告 + 一个数据录入表单。

| 模板 | 路径 | 用途 |
|------|------|------|
| 每日仪表盘 | `assets/daily-dashboard.html` | 单日热量、运动、蛋白质全貌（只读报告） |
| 周度趋势报告 | `assets/weekly-report.html` | 7 天体重/缺口/蛋白质/步数趋势（只读报告） |
| 周数据追踪表 | `assets/weekly-tracker.html` | 可填写的 HTML 表单，录入 7 天原始数据 |

## 触发条件

用户说以下内容时自动生成 HTML 可视化：

- "生成日报""可视化今天的""看看今天的图表""生成 HTML"
- "周报""本周趋势""可视化这周""生成周报告"
- 用户先完成每日热量分析后，主动追问"能不能可视化一下"

## 使用流程

### 步骤 1：读取模板

读取对应模板文件的完整内容：

```
Read: assets/daily-dashboard.html  （或 weekly-report.html）
```

### 步骤 2：替换 JSON 数据

模板中有一段 `<script type="application/json">` 标签，内含示例数据。用当天实际数据替换整个 JSON 对象。

**关键规则：**
- 只替换 JSON 内容，不要修改 HTML 结构、CSS、JavaScript 渲染逻辑
- 保持 JSON 字段名不变，缺少数据的字段填 `null` 或 `0`
- 确保替换后的 JSON 是合法的（无注释、无尾逗号）

### 步骤 3：写入输出文件

将修改后的完整 HTML 写入用户工作目录：

```
输出路径：{工作目录}/fitness-dashboard-{YYYY-MM-DD}.html
       或：{工作目录}/fitness-weekly-{YYYY-MM-DD}.html
```

### 步骤 4：展示文件

使用 `present_files` 工具将 HTML 文件路径传给用户，自动在浏览器面板中预览。

---

## 每日仪表盘数据结构

```json
{
  "date": "2025-07-16（三）",
  "goal": "减脂",
  "goalType": "cut",           // cut | bulk | maintain
  "stats": {
    "tdee": 2290,
    "intake": 1700,
    "deficit": -590,            // 负=赤字, 正=盈余, 0=持平
    "protein": 125,
    "proteinTarget": 122,
    "steps": 8000,
    "weight": 70.5,
    "bodyFat": 18.2,             // 体脂率(%)，无数据填 null
    "bmr": 1650,
    "bmrSource": "估算"          // "实测" 或 "估算"
  },
  "tdeeBreakdown": {
    "bmr": 1815,                // BMR × 1.1
    "neat": 360,                // 步数 × 45 / 1000
    "eat": 280,                 // Σ(MET × 体重 × 小时)
    "labels": ["静态保本 (BMR×1.1)", "日常活动 (NEAT)", "运动消耗 (EAT)"],
    "values": [1815, 360, 280]
  },
  "meals": [
    { "name": "早餐", "calories": 400, "items": "燕麦粥 + 鸡蛋 x2" },
    { "name": "午餐", "calories": 600, "items": "鸡胸肉 + 米饭 + 西兰花" },
    { "name": "晚餐", "calories": 500, "items": "牛排 + 沙拉" },
    { "name": "加餐", "calories": 200, "items": "蛋白粉 + 香蕉" }
  ],
  "exercises": [
    { "type": "力量训练", "met": 4.0, "duration": 50, "calories": 233 },
    { "type": "散步", "met": 3.0, "duration": 30, "calories": 105 }
  ]
}
```

### 图表说明

| 图表 | 类型 | 展示内容 |
|------|------|---------|
| TDEE 三层拆解 | 水平堆叠柱状图 | BMR×1.1 / NEAT / EAT 三层占比 |
| 摄入 vs 消耗 | 柱状图 | TDEE 与全天摄入并排对比 |
| 餐次热量分布 | 环形图 | 各餐热量占比 + 百分比 |
| 蛋白质达成 | SVG 进度环 | 达成率 + 实际/目标/差距/每公斤 |
| 运动消耗明细 | 表格 | 每项运动的 MET、时长、消耗 + 合计 |

---

## 周度趋势报告数据结构

```json
{
  "weekRange": "2025-07-10 ~ 2025-07-16",
  "goal": "减脂",
  "goalType": "cut",
  "proteinTarget": 122,
  "summary": {
    "avgDeficit": -420,
    "avgProtein": 118,
    "weightChange": -0.8,
    "trainingDays": 4,
    "avgSteps": 8500,
    "bodyFat": 18.2
  },
  "days": [
    {
      "day": "周一",
      "date": "07-10",
      "weight": 71.0,
      "bodyFat": null,
      "intake": 1800,
      "tdee": 2200,
      "deficit": -400,
      "protein": 115,
      "steps": 7000,
      "trained": true
    }
    // ... 7 天数据
  ],
  "insights": [
    { "type": "good", "text": "本周体重下降 0.8kg，减脂节奏稳定" },
    { "type": "warn", "text": "周六出现热量盈余 +50 kcal，注意周末饮食控制" },
    { "type": "info", "text": "训练 4 天，建议下周保持或增加至 5 天" }
  ]
}
```

### 图表说明

| 图表 | 类型 | 展示内容 |
|------|------|---------|
| 体重趋势 | 折线图 | 7 天体重变化曲线 |
| 每日热量缺口 | 柱状图 | 绿=赤字, 黄=盈余, 灰=持平 |
| 蛋白质达成率 | 柱状图+目标线 | 每日实际摄入 vs 目标虚线, 颜色按达成率 |
| 步数趋势 | 折线图 | 7 天步数变化 |
| 每日明细 | 表格 | 体重/摄入/TDEE/缺口/蛋白质/步数/训练 |
| 本周洞察 | 列表 | 好✅ 警告⚠️ 信息💡 标记的要点 |

### 洞察生成规则

- `good`：达成目标、趋势向好、数据亮眼
- `warn`：偏离目标、异常波动、需要关注
- `bad`：严重偏离、连续异常
- `info`：中性信息、建议

洞察数量 3-5 条，按重要性排序。遵循"数据说话"原则——只呈现事实，不评价用户。

---

## 缺失数据处理

| 场景 | 处理方式 |
|------|---------|
| 无步数 | `steps: 0`，TDEE 中 NEAT 设 0，标注"步数待补" |
| 无体重 | `weight: null`，蛋白质每公斤项不显示 |
| 无运动 | `exercises: []`，EAT 设 0，运动表显示"今日无训练" |
| 餐次不全 | 只填有的餐次，其余省略 |
| 无体脂率 | 不影响 HTML 渲染，体脂率不显示 |

## 技术说明

### 模板基础

- 模板使用 Chart.js 4.4.1（CDN 加载，需要联网）
- 页面为纯静态 HTML，无需后端
- 响应式布局，支持移动端查看
- 所有图表可交互（悬停显示数值、图例可切换）

### 双主题设计系统

模板内置 **深色健身力量感 + 日间清爽风** 双主题，右上角 ☀/🌙 图标一键切换。

#### CSS 变量体系

两套主题通过 `[data-theme="dark"]` 和 `[data-theme="light"]` 属性选择器定义 CSS 变量，所有颜色、阴影、边框均联动切换：

| 变量类别 | 深色主题 | 日间主题 |
|---------|---------|---------|
| 画布背景 `--bg` | `#1C1C1E` | `#F5F5F7` |
| 卡片背景 `--bg-card` | `#2C2C2E` | `#FFFFFF` |
| 图表内区 `--bg-inner` | `#1C1C1E` | `#F0F0F2` |
| 主文字 `--text` | `#FAFAFA` | `#1C1C1E` |
| 辅助文字 `--text-secondary` | `#A1A1AA` | `#6E6E73` |
| 暗文字 `--text-muted` | `#71717A` | `#8E8E93` |
| 边框 `--border` | `#27272A` | `#E5E5EA` |
| 强边框 `--border-strong` | `#3F3F46` | `#D1D1D6` |
| 红色 `--accent-red` | `#FF375F` | `#E5243D` |
| 绿色 `--accent-green` | `#30D158` | `#24A853` |
| 青色 `--accent-blue` | `#00C7BE` | `#009E96` |
| 橙色 `--accent-orange` | `#FF8F3E` | `#D97A2E` |
| 紫色 `--accent-purple` | `#7C3AED` | `#6D28D9` |
| 红色发光 `--glow-red` | `rgba(255,55,95,0.4)` | `rgba(229,36,61,0.25)` |
| 绿色发光 `--glow-green` | `rgba(48,209,88,0.4)` | `rgba(36,168,83,0.25)` |
| 青色发光 `--glow-blue` | `rgba(0,199,190,0.4)` | `rgba(0,158,150,0.25)` |
| 橙色发光 `--glow-orange` | `rgba(255,143,62,0.4)` | `rgba(217,122,46,0.25)` |
| 紫色发光 `--glow-purple` | `rgba(124,58,237,0.4)` | `rgba(109,40,217,0.25)` |
| 卡片阴影 `--shadow-card` | `rgba(0,0,0,0.35)` | `rgba(0,0,0,0.06)` |
| 蛋白质环底色 `--protein-ring-bg` | `#3F3F46` | `#E5E5EA` |

#### 主题切换机制

1. **UI 按钮**：右上角 `.theme-toggle` 按钮，深色模式显示 ☀ + "日间"，日间模式显示 🌙 + "夜间"
2. **持久化**：点击后写入 `localStorage('fitness-theme')`，下次打开自动恢复
3. **Chart.js 重建**：切换时销毁所有 Chart.js 实例 → 用新主题色重新渲染（网格线、坐标轴、tooltip、数据标签颜色全部联动）
4. **SVG 环**：蛋白质进度环使用 CSS 变量（`var(--protein-ring-bg)`），自动跟随主题
5. **发光效果**：深色模式启用 `text-shadow` 发光和 `filter: drop-shadow()`；日间模式关闭所有发光（`[data-theme="light"]` 覆盖为 `none`）

#### 字体系统

- 展示数字：**Space Grotesk** Bold 700（英文字母间距 -0.02em，`font-variant-numeric: tabular-nums`）
- 中文标题/正文：**Noto Sans SC** Bold/Medium/Regular
- 英文正文：**IBM Plex Sans** Medium
- 回退：PingFang SC → Microsoft YaHei → sans-serif

#### 视觉细节

| 元素 | 深色模式 | 日间模式 |
|-----|---------|---------|
| 标题文字 | 发光阴影 `text-shadow` | 无阴影 |
| KPI 数字 | 发光阴影（绿色/红色/蓝色/橙色/紫色按语义） | 无阴影 |
| 卡片边框 | `1px solid #27272A` | `1px solid #E5E5EA` |
| 卡片圆角 | 20px | 20px（两主题共享） |
| MET 徽章 | 半透明背景 + 高饱和文字 | 低透明度背景 + 深色文字 |
| 目标徽章 | 内发光 + 外发光 | 无发光，纯色边框 |
| 卡片悬停 | `translateY(-2px)` + 加深阴影 | 同动效，更轻阴影 |
| 主题按钮悬停 | 图标旋转 30° | 图标旋转 -30° |

#### 设计来源

设计原型基于 Ardot 设计平台构建：

- **设计文件**：`健身热量仪表盘设计优化`（Ardot ID: 704440990413193）
- **风格组合**：soft-apple-fitness + brutalism-grotesk + hook-mega-number
- **设计变量**：Semantic 色彩系统 + Primitives 圆角/间距/字体系统

> 如需修改设计原型，可在 Ardot 中编辑上述设计文件，然后同步更新 HTML 模板中的 CSS 变量。

---

## 周数据追踪表（weekly-tracker.html）

### 用途

可填写的 HTML 表单，用户在浏览器中录入 7 天原始数据。替代手写 Markdown 追踪文件的工作流。

### 表单结构

**顶部设置栏（用户画像）：**
- 目标（减脂/增肌/维持）
- 体重、BMR、BMR 来源、蛋白质目标
- 周日期范围（自动推算 7 天日期）

**7 天 tab 切换，每天包含：**
- 运动：MET 下拉选择 + 时长输入，自动计算消耗（MET × 体重 × 时长）
- 步数：输入后自动计算 NEAT（步数 × 45 / 1000）
- 四餐：早餐/午餐/晚餐/加餐，食物描述（必填）+ 热量（可选，留空则 AI 估算）
- 身体数据：体重、体脂率、腰围
- 恢复：睡眠时长、睡眠质量
- 备注：文字

**底部实时状态栏：**
- 摄入 / TDEE / 缺口 / 蛋白质目标（随当天填写实时更新）

### 数据存储

- **localStorage**：每次输入自动保存（key: `fitness-tracker-data`），刷新不丢。保存失败时（照片撑爆）自动降级为剥离照片保存
- **内嵌 JSON**：数据同步写入 `<script type="application/json" id="trackerData">` 标签
- **本地目录**（File System Access API）：点击"日报"或"周报"时，数据写入用户配置目录下的 `每周追踪 {周开始}~{周结束}_{YYYYMMDD-HHmm}.md`（不覆盖历史文件），照片写入 `{日期}-{餐次}.jpg`（如 `2026-07-13-早餐.jpg`）。目录权限持久化到 IndexedDB

### 底部四个按钮

| 按钮 | 功能 | 输出 |
|------|------|------|
| 📄 导出 MD | 下载 Obsidian 兼容的 Markdown 文件 | `每周追踪 YYYY-MM-DD~YYYY-MM-DD.md` |
| 📥 更新数据 | 从保存目录读取最新 MD，解析 `tracker-data` JSON 块回填表单 | 表单数据更新 |
| 📊 日报 | 尝试 FSAPI 保存（写后验证），剪贴板始终含 `<tracker-data>` JSON | 文件模式：`每周追踪 *.md` + `{日期}-{餐次}.jpg` + 剪贴板="日报\\n当前查看：周X YYYY-MM-DD\\n保存目录：xxx\\n食物照片：xxx.jpg\\n<tracker-data>JSON(无photo)</tracker-data>"；降级模式：剪贴板="日报\\n当前查看：周X YYYY-MM-DD\\n<tracker-data>JSON(含photo base64)</tracker-data>" |
| 📈 周报 | 同上，触发词为"周报" | 同上，触发词改为"周报"，日报特有"当前查看"行省略 |

### 顶部"🔄 新的一周"按钮

| 按钮 | 功能 | 行为 |
|------|------|------|
| 🔄 新的一周 | 保存当前周数据 + 开始新的一周 | 确认后先保存当前 MD → weekStart +7 天 → 清空日数据（保留 profile）→ 重新生成日期 |

**日报/周报按钮的两种模式（自动检测，写后回读验证）：**
- **文件模式**（Chrome/Edge 等真实浏览器）：File System Access API 写入后回读验证通过 → 照片存为 `{日期}-{餐次}.jpg` 到磁盘。剪贴板含触发词 + 保存目录路径 + 照片文件名列表 + `<tracker-data>` JSON（照片 base64 已剥离）。AI 按文件名从目录读取照片。**保存目录路径需为完整本地路径**（如 `E:\Myworkbuddy\测试`），FSAPI 仅返回文件夹名，用户需手动补全；路径不完整时输入框标黄警告，剪贴板中提示"路径不完整"
- **降级模式**（WorkBuddy 内嵌浏览器等，FSAPI 不报错但实际未写入磁盘）：写后回读验证失败 → 剪贴板含触发词 + `<tracker-data>` JSON（**照片 base64 保留在 `photo` 字段中**）。AI 解码 base64 → 保存为 .jpg 文件 → Read 工具读取查看

### JSON 数据结构

```json
{
  "profile": {
    "weight": 68,
    "bmr": 1605,
    "bmrSource": "估算",
    "goal": "增肌",
    "proteinTarget": 122
  },
  "weekStart": "2026-07-06",
  "weekEnd": "2026-07-12",
  "days": [
    {
      "day": "周一",
      "date": "2026-07-06",
      "exercises": [
        { "type": "力量训练", "met": 4.0, "duration": 40 }
      ],
      "steps": 9603,
      "meals": [
        { "name": "早餐", "items": "KFC能量碗", "calories": 651, "photo": "data:image/jpeg;base64,..." },
        { "name": "午餐", "items": "鸡腿饭", "calories": 700, "photo": null },
        { "name": "晚餐", "items": "卤鸡腿+煎蛋+虾肉砂锅", "calories": 810, "photo": null },
        { "name": "加餐", "items": "", "calories": null, "photo": null }
      ],
      "weight": 68.3,
      "bodyFat": 18.2,
      "waist": null,
      "sleep": 6,
      "sleepQuality": "一般",
      "notes": ""
    }
  ]
}
```

**注意：** 保存为 MD 文件时，有照片的餐次在内容后标记 `📷[日期-餐次.jpg]`（如 `📷[2026-07-13-早餐.jpg]`），照片单独存为 `{日期}-{餐次}.jpg`。tracker-data JSON 块中对应餐次会包含 `hasPhoto: true` 和 `photoFile: "2026-07-13-早餐.jpg"` 字段。AI 读取照片时按 `photoFile` 文件名从同目录读取对应的 `.jpg` 文件。MD 文件末尾包含 ` ```tracker-data ` JSON 代码块，供 AI 和表单双向读写。

### AI 解析流程

**统一流程**（用户粘贴剪贴板内容，始终含 `<tracker-data>` JSON）：

1. 从用户消息中提取 `<tracker-data>` 标签内的 JSON
2. 检查照片数据（两种情况）：
   - **JSON 中 `photo` 字段有值**（`data:image/jpeg;base64,...`）→ 降级模式，照片未存到磁盘。AI 应：①提取 base64 ②用 Python/Node 脚本保存为 `outputs/{photoFile}` ③用 Read 工具读取该 .jpg 文件查看照片 ④估算热量
   - **JSON 中 `hasPhoto: true` 但 `photo` 为空**，且有"保存目录：xxx" → 文件模式，照片已存到磁盘。AI 按 `photoFile` 文件名从保存目录读取照片文件。**路径处理：** 若"保存目录"为完整路径（含盘符如 `E:\...`）直接使用；若仅有文件夹名或提示"路径不完整"，需全盘搜索该文件夹名定位实际路径
3. 遍历每天数据，对每天计算：
   - TDEE = `profile.bmr × 1.1` + `steps × 45 / 1000` + `Σ(MET × weight × duration / 60)`
   - 摄入 = `Σ(meals.calories)`（null 项不累加，如有食物描述但无热量，AI 估算）
   - 缺口 = 摄入 - TDEE
   - 蛋白质：如用户未单独提供，从食物描述估算
4. 根据触发词选择模板：
   - "周报" → 组装全周数据，注入 `weekly-report.html` 生成周度趋势报告
   - "日报" → 提取"当前查看"指定日期的单日数据，注入 `daily-dashboard.html` 生成当日仪表盘
5. 如需回写热量（文件模式），执行下方回写流程

**AI 回写数据到 MD（热量估算后）：**

1. 读取原始 MD 文件内容
2. 提取 ` ```tracker-data ` JSON 代码块，解析为对象
3. 将估算的热量填入对应餐次的 `calories` 字段（如 `days[0].meals[0].calories = 450`）
4. 同步更新 MD 表格中对应餐次的热量列（`| 早餐 | 鸡胸肉沙拉 📷[2026-07-13-早餐.jpg] | ~450 |`）
5. 将更新后的 JSON 重新序列化，替换原代码块，写回 MD 文件
6. 告知用户"已更新数据，请在表单中点击「更新数据」加载"

**用户回填流程：**

1. 用户在表单中点击"更新数据"
2. 表单通过 File System Access API 读取目录下最新的 `.md` 文件
3. 解析 ` ```tracker-data ` JSON 代码块
4. 回填到表单（保留当前已有的照片数据不被覆盖）
5. 如某餐次有 `hasPhoto: true` 和 `photoFile` 但内存中无 base64 照片，自动从目录读取对应 `{日期}-{餐次}.jpg` 文件恢复照片
6. 重新渲染表单和实时统计

**降级模式补充说明**（剪贴板消息含 `<tracker-data>` JSON 且 `photo` 字段有 base64 数据）：

1. 从 `<tracker-data>` 标签中提取 JSON
2. 如触发词为"日报"，从"当前查看"行提取目标日期，聚焦该天数据
3. 照片处理：JSON 中 `photo` 字段含 `data:image/jpeg;base64,...`，AI 应：
   - 提取 base64 字符串
   - 用 Python 脚本保存为 `outputs/{photoFile}`（如 `outputs/2026-07-13-早餐.jpg`）
   - 用 Read 工具读取该 .jpg 文件查看照片
   - 估算食物热量
4. 同上述步骤 3-5 计算和生成报告

### 与 Obsidian 工作流的兼容

| 场景 | 操作 |
|------|------|
| 用户已有 Obsidian MD 文件 | 直接 `@文件路径` 给 AI，AI 解析 MD 格式 |
| 用户用 HTML 表单录入（文件模式） | 点"📊 日报"或"📈 周报" → 切回对话粘贴剪贴板内容（含保存目录+照片文件名+JSON） → AI 读 MD + 按文件名读照片生成报告 → AI 回写热量到 MD → 表单点"更新数据"回填 |
| 用户用 HTML 表单录入（降级模式） | 点"📊 日报"或"📈 周报" → 切回对话粘贴数据（含 JSON+照片 base64） → AI 解析 JSON → 解码 base64 保存为 .jpg → Read 查看照片 → 估算热量生成报告 |
| 用户导出 MD 后继续用 Obsidian | 点击"导出 MD" → 在 Obsidian 中打开 → 需要时 `@文件` |
| 混合使用 | 表单录入 → 导出 MD → Obsidian 手动补充 → `@文件` 给 AI |
