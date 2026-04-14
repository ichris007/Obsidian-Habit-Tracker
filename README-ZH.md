# Obsidian-Habit-Tracker

## 📋 功能概述

这是一个运行在 Obsidian Datacore 插件中的**习惯追踪器**，可以将每日习惯记录直接保存在每日笔记的 frontmatter 中，并提供可视化的追踪界面。

在[此代码](https://forum.obsidian.md/t/building-a-powerful-habit-tracker-in-obsidian-a-complete-guide/92884)基础上修改：
- 拆分js和CSS代码
- 适配深色/浅色主题
- 修改habits frontmatter 为数组格式，并兼容之前的对象格式

### 核心功能

1. **习惯打卡** - 点击习惯卡片切换完成/未完成状态
2. **数值记录** - 为每个习惯记录具体数值（页数、分钟、英里、小时）
3. **日历视图** - 显示最近 6 天的习惯完成情况
4. **统计分析** - 查看习惯趋势、月度/年度目标进度
5. **历史数据** - 浏览和编辑所有历史记录

### 界面展示

<div align="center">
  
#### 日历视图
<img src="screenshot/Habit%20Tracker%201.jpg" alt="Calendar View showing 6 days of habits" width="600" />

#### 趋势统计  
<img src="screenshot/Habit%20Tracker%202.jpg" alt="Statistics view with goals and metrics" width="600" />

#### 历史视图
<img src="screenshot/Habit%20Tracker%203.jpg" alt="Historical data table with pagination" width="600" />

</div>

---

## 🎯 追踪的六大习惯

| 习惯 | 标识 | 单位 | 默认值 |
|------|------|------|--------|
| 📚 阅读 | `reading` | pages | 25 页 |
| 🧘‍♂️ 冥想 | `meditation` | minutes | 20 分钟 |
| 🏋️ 力量训练 | `weightlifting` | minutes | 45 分钟 |
| 🏃‍♂️ 跑步 | `run` | miles | 2 英里 |
| ✍️ 写作 | `writing` | minutes | 30 分钟 |
| 😴 睡眠 | `sleep` | hours | 8 小时 |

### 目标设定

- **月度完美日目标**：20 天（所有习惯都完成）
- **年度完美日目标**：250 天

---

## 📁 数据存储格式

### 文件位置
```
Notes/Daily Notes/YYYY-MM-DD.md
```

### Frontmatter 格式

```yaml
---
created: 2026-04-14
tags:
  - daily
habits:
  - reading: true
  - reading_duration: 25
  - meditation: true
  - meditation_duration: 20
  - weightlifting: true
  - weightlifting_duration: 45
  - writing: true
  - writing_duration: 30
  - run: true
  - run_duration: 2
  - sleep: true
  - sleep_duration: 8
---
```

### 数据说明

- **习惯状态**：`true` 表示完成，`false` 表示未完成
- **数值字段**：格式为 `{习惯ID}_duration`
- **数组格式**：habits 使用数组格式存储（已支持向后兼容对象格式）

---

## 🎨 界面功能详解

### 1. 顶部导航栏
- 显示当前日期
- 左右箭头切换日期
- 日期格式：`MMMM dd, yyyy`（如 April 14, 2026）

### 2. 日历视图（主视图）
- 显示最近 6 天（今天 + 前 5 天）
- 3x2 网格布局
- 每个卡片显示：
  - 星期和日期
  - 6 个习惯按钮（点击切换状态）
  - 完成进度条和百分比
- 选中日期有高亮边框

### 3. 统计视图（📊 按钮）
- **目标进度卡片**：
  - 月度完美日进度
  - 年度完美日进度
  - 当前连续完美天数
- **习惯详细统计**（每个习惯单独卡片）：
  - 最近 30 天总量
  - 年初至今（YTD）总量
  - 趋势变化百分比（↑/↓）

### 4. 历史数据视图（📚 按钮）
- 表格显示所有每日笔记
- 每页 20 条记录，支持翻页
- 可直接在表格中编辑：
  - 点击 ✓/✗ 切换习惯状态
  - 点击数值修改具体数量
- 显示每日完成百分比

### 5. 交互功能
- **切换习惯**：点击习惯卡片即可切换完成状态
- **修改数值**：点击已完成的数值，输入新数值后按回车或失焦保存
- **实时保存**：所有修改立即写入 frontmatter

---

## 🚀 安装与使用

### 安装方法1：下载Habit Tracker Vault

<img src="screenshot/下载Habit%20Tracker%20Vault.jpg" alt="Calendar View showing 6 days of habits" width="600" />

下载ZIP文档后解压，用Obsidian打开。

### 安装方法2：手动配置

#### 前置要求

1. **Obsidian** 已安装
2. **Datacore 插件**已安装并启用
3. 存在 `Notes/Daily Notes/` 文件夹

#### 安装步骤

1. 创建 `habit-tracker.css` 文件，粘贴 CSS 代码
2. 创建 `habit-tracker.md` 文件，粘贴 JS 代码

````markdown
```datacorejsx

[粘贴 JS 代码]
```
````

### 首次使用

1. 确保有每日笔记文件
2. 点击习惯卡片开始打卡
3. 数值会自动填充默认值
4. 点击数值可以修改为实际值

---

## ⚙️ 自定义配置

### 修改习惯列表

编辑 `HABITS` 常量：

```javascript
const HABITS = [
  { id: 'reading', emoji: '📚', label: 'Pages', defaultDuration: 25, unit: 'pages' },
  // 添加新习惯
  { id: 'newhabit', emoji: '🎯', label: 'New Habit', defaultDuration: 10, unit: 'minutes' }
];
```

### 修改目标值

编辑 `GOALS` 常量：

```javascript
const GOALS = {
  perfectDays: {
    monthly: 20,  // 修改月度目标
    yearly: 250   // 修改年度目标
  }
};
```

### 修改日期范围

在 `CalendarView` 组件中修改循环次数：

```javascript
for (let i = 0; i < 6; i++) {  // 6 = 显示天数
  dates.push(currentDate.minus({ days: i }));
}
```

### 修改每页显示数量

在 `HistoricalView` 组件中：

```javascript
const itemsPerPage = 20;  // 改为想要的数量
```

---

## 🎨 主题适配

代码使用 Obsidian 的 CSS 变量，**自动适配**深色/浅色主题：

- `var(--background-primary)` - 主背景色
- `var(--background-secondary)` - 次要背景色
- `var(--text-normal)` - 普通文本
- `var(--interactive-accent)` - 强调色
- `var(--color-green/red/yellow)` - 进度条颜色

切换 Obsidian 主题时，界面颜色会自动跟随变化。

---

## 💡 使用技巧

### 批量编辑历史数据
使用历史数据视图可以快速修改多天的习惯记录，无需打开每个笔记文件。

### 查看年度趋势
通过统计视图的 YTD 数据，可以了解年度习惯完成情况。

### 完美日追踪
完成所有 6 个习惯即可获得"完美日"，系统会自动统计并显示在目标进度中。

### 数值记录
- 阅读：记录实际阅读页数
- 运动：记录运动时长（分钟）
- 跑步：记录里程（英里）
- 睡眠：记录小时数

---

## ⚠️ 注意事项

1. **文件路径**：确保每日笔记在 `Notes/Daily Notes/` 路径下
2. **文件命名**：必须使用 `YYYY-MM-DD` 格式
3. **Datacore 插件**：需要启用并正确配置
4. **数组格式**：新数据会保存为数组格式，旧数据自动兼容
5. **性能**：大量历史数据时，分页浏览会更流畅

---

## 📝 示例工作流

1. **早晨**：打开习惯追踪器，点击已完成习惯（如睡眠）
2. **白天**：完成运动后，点击并修改实际时长
3. **晚上**：记录阅读页数、冥想时长等
4. **周末**：查看统计视图，分析本周完成情况
5. **月末**：检查月度完美日目标进度

---

## 🐛 常见问题

**Q1: 点击习惯没有反应？**

A: 检查文件路径是否正确，确保笔记文件存在。

**Q2: 修改数值后没有保存？**

A: 按回车键或点击其他地方让输入框失去焦点。

**Q3: 历史数据不显示？**

A: 检查每日笔记的 frontmatter 格式是否正确。

**Q4: 深色模式下颜色不对？**

A: 确保使用了分离的 CSS 文件或代码中包含完整的样式。

---
**享受你的习惯追踪之旅！** 🎯
