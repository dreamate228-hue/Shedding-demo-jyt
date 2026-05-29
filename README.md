# Shedding Mobile Demo

Shedding 是一个面向海外中年健康方向的移动端原型 Demo。当前项目把 Figma 设计稿实现为单文件 HTML 原型，包含 Today、Move、Track 三个底部 tab 页面，用于演示健康状态记录、轻运动课程、饮食摄入和身体指标追踪等核心体验。

项目目标是尽量贴近 Figma 视觉稿，Demo 以视觉还原和交互演示为主，不连接后端。

## Quick Start

直接在浏览器打开：

```text
index.html
```

这是一个 standalone HTML 文件，不需要安装依赖或启动构建工具。页面内资源主要来自本地 `assets/` 目录。

## Project Structure

```text
.
├── index.html                         # 单文件 Demo，包含 HTML/CSS/JS
├── assets/                            # 图标、课程图、水母分层素材、Track 卡片素材等
├── PLAN.md                            # 产品 PRD、用户洞察和设计策略
├── README.md                          # 项目说明和交接文档
├── 液态玻璃参考/                      # 用户提供的液态玻璃参考代码
└── 【睿迄科技】全栈设计师5-7天挑战.pdf # 原始挑战说明 PDF
```

根目录里的 `move.png`、`track.png`、`qa15-*.png`、`today-*.png` 等 PNG 是历史 QA/对比截图，不是运行依赖，可以删除。`.chrome-track-test*` 也是本地浏览器测试临时目录，可以删除。

## Product Direction

完整产品方向见 `PLAN.md`。README 只保留后续实现窗口最需要的上下文。

Shedding 的定位是面向欧美 `40-60` 岁女性的 AI native 轻运动减重 App。目标用户可能经历过节食、卡路里追踪、高强度训练等失败体验，也可能有膝盖、腰背、关节、精力下降等身体限制。

一句话价值主张：

```text
Feel lighter through gentle, adaptive movement.
```

当前 Demo 中：

- Today 对应 Ease In：低压力开始当天状态和建议。
- Move 保留 Move：探索身体友好的课程。
- Track 承担 Release 的一部分：展示饮食、身体指标和轻量追踪。

## Design Source

Figma file key:

```text
saLZV6eXEtaoVJhKRgMhyM
```

本轮和历史实现重点参考过的节点：

- Today 初始页：`1000:2182`
- Today 结果页：`969:6659`
- Move 页：`969:7964`
- Track 页：`969:7383`
- Track Diet 主卡旧版：`1058:6992`
- Track Diet 环形图/水母/食物条细节：`1058:6992`, `1059:7198`, `1066:7256`, `1067:7325`
- Track Diet 异形卡和模式切换：`1075:7594`, `1075:7595`, `1084:7683`, `1084:7671`
- Today plan list：`989:11688`
- Bottom tab states：`993:11923`

## Current Implementation Scope

当前 `index.html` 实现了：

- Today 页
  - 初始 mood 选择
  - 选择 mood 后切换到结果卡片
  - AI 文案打字机效果
  - 推荐课程卡片渐入
  - My Plan 列表
  - More details 跳转 Track
- Move 页
  - 分类 chip
  - 课程列表
  - Continue 进度卡
  - 点击课程后显示 Continue 卡和骨架屏过渡
- Track 页
  - 日期条
  - Diet/Weight 模式切换卡片
  - Diet 饮食摄入记录、环形图、气泡、水母眼神跟随
  - 每个日期独立保存饮食小卡和 intake 数值
  - 指标卡片网格
- 全局
  - 底部 tab 切页
  - iPhone 状态栏和 home indicator
  - 移动端短视口适配

## Track Diet Card Notes

本窗口主要集中修改 Track 页第一张 Diet 大卡片。

### 异形卡片结构

当前卡片使用新 Figma 规范：

```text
.diet-card
├── .diet-blue-card          # 蓝色底卡，四边相对外层内缩 6px
├── .diet-mode-button        # 右上 Diet/Weight 切换热区
└── .diet-shell
    ├── .diet-top            # 左上 Remaining 白色区域
    ├── .diet-corner         # assets/diet-card-corner.svg
    └── .diet-content        # 下方内容区，保留原 Diet 内容
```

关键视觉参数：

- 蓝色底卡：`.diet-blue-card { inset: 6px; border-radius: 20px; }`
- 右上切换热区：实际按钮 `95px × 28px`，伪元素扩展为 `130px × 44px` 热区。
- Diet/Weight 文案字重：`500`。
- 文案槽宽：`63px`，icon 宽 `20px`，二者间距 `12px`。
- 右上按钮当前为了视觉对齐设置为 `right: 24px; top: 15px;`，相对蓝色卡上边视觉距离约 `9px`。
- Remaining 行强制单行，`Remaining` 与数值间距 `8px`，数值与 `kcal` 间距 `4px`。

Weight 模式暂未设计，点击右上按钮切换到 Weight 后，内容区仅展示：

```text
暂未设计，敬请期待
```

再次点击切回 Diet，原 Diet 内容恢复。

### 日期记录逻辑

每个可点击日期都有独立饮食记录：

- 初始 intake 为 `0`，remaining 为 `600`。
- 点击 Add 会随机添加一张食物小卡，同时累计该日期的 intake。
- 切换到其他过去日期时，该日期从空记录开始。
- 已添加过的日期会保存该日期的小卡列表和 intake，切回时恢复。
- 未来日期不可点击。

相关状态：

```js
selectedDateKey
dietRecords = {
  [dateKey]: { intake, foods }
}
```

### 环形进度

环形图不再沿用设计稿布尔/遮罩旋转方式，而是用 SVG path 描边：

- 背景和进度共用同一条圆弧 path。
- 进度值为 `intake / 600`，进度长度在 600 后封顶。
- 超过 600 后颜色继续从蓝青渐变过渡到红粉，`1200 kcal` 时达到完整红粉。
- 当前渐变方向：
  - 正常：前段青色，末端蓝色。
  - 超量：前段粉色，末端深红。
- 气泡使用 5 个点，参考 Figma 节点 `1066:7256`。
- 气泡取进度末端沿路径回退 `8px` 的点，外法线偏移 `2px`。
- 气泡动画由 JS 逐帧沿 SVG path 取点和切线，不再用 CSS 直线插值，避免径向抖动。

### 小水母分层

水母已改为分层素材：

- `assets/jelly-body.svg`
- `assets/jelly-eye-white-left.svg`
- `assets/jelly-eye-white-right.svg`
- `assets/jelly-pupils.svg`
- `assets/jelly-shadow-new.svg`

DOM 分层：

```text
.jelly
├── img.body
└── .jelly-eyes
    ├── img.jelly-eye-white.left
    ├── img.jelly-eye-white.right
    └── img.jelly-pupils
```

眼神状态：

- `intake = 0` 为失焦状态：眼睛保持偏右，眼珠略向上/偏右下调后的稳定位置。
- `intake > 0` 为聚焦状态：眼睛层基准与面部居中对齐，然后整体跟随进度末端运动；眼珠作为眼睛层子集继续做小幅跟随。
- 眼睛整体幅度较大，接近脸颊边缘；眼珠层有独立基准修正，避免向右看时超出眼白。

水母横向居中：

- `.jelly { left: 50%; margin-left: -54.5px; }`
- `.jelly-shadow { left: 50%; margin-left: -34px; }`

水母浮动：

- 原本上浮 `18px`，本窗口改为 `9px`。
- 最低点保持不变。
- 影子动画未跟随改动。

### 食物小卡和 Add

底部食物条：

- 空态时 Add 按钮填满整条区域。
- 添加第一张小卡时动画顺序：
  1. Add 文字和 icon 淡出。
  2. Add 从宽态向右收缩。
  3. 窄态文字和 icon 淡入。
  4. 总时长约 `0.66s`。
- 后续添加的小卡会从右侧轻微滑入淡显。
- 左侧滚动区为填充宽度，右侧窄 Add 固定 `36px`。
- 右侧白色蒙层只在右侧还有可滚动内容时出现；滚到最右侧后消失。
- 左侧白色蒙层只在已经向右滚动、左侧有被遮住内容时出现。

## Other Key Notes

### Top Sticky Areas

Today、Move、Track 顶部固定区域统一使用：

```text
.top-sticky
├── .statusbar
└── .top-row 或 .page-title-row
```

位置要求：

- 状态栏：`top = 0`, `height = 59`
- 下方头像/标题行：`top = 67`
- Today 头像行高度约 `40`
- Move/Track 标题行高度约 `48`

注意：

- 不要再给顶部固定区域加 `backdrop-filter`，之前用户反馈会产生割裂感。
- 不要恢复旧的 `.page-title-row::after` 白色蒙层。

### Bottom Tab / Liquid Glass

底部 tab 使用 JS 生成 displacement map，实现近似 iOS 26 液态玻璃：

- 主要函数：`buildBottomTabLiquidFilters()`
- 使用 `feImage + feDisplacementMap`
- 三个 filter：
  - `bottom-tab-liquid-filter`
  - `bottom-tab-rim-filter`
  - `bottom-tab-highlight-filter`

如后续继续微调液态玻璃，优先小步调整这些位置：

- `buildBottomTabLiquidFilters()` 中的 `edgeWidth`, `edgeStrength`, `coreStrength`
- `.liquid-tabs` 的 `backdrop-filter`、外圈高光透明度
- `.tab-highlight` 的填充透明度、blur、边缘效果

## Verification Notes

本窗口使用本地 Chrome headless + DevTools Protocol 做过多轮验证，确认：

- Track Diet 日期记录可以按日期保存和恢复。
- 过去日期初始为 `0 / 600`，可添加小卡。
- Diet/Weight 切换可点击，热区命中 `#dietModeToggle`。
- Weight 模式显示占位文案，Diet 模式恢复内容。
- Remaining 行不换行，间距为 `8px / 4px`。
- Intake/Burned 与食物小卡不重叠。
- 气泡沿 path 运动，不再使用 CSS 直线插值。
- 水母和影子在宽机型中横向居中。

## Current Git Notes

当前工作区可能仍有一些与本窗口无关的资产删除状态。处理 git 前请先查看：

```text
git status --short
```

不要随意恢复或删除其他窗口留下的变更。

本窗口新增/更新过的关键资产：

- `assets/diet-card-corner.svg`
- `assets/jelly-body.svg`
- `assets/jelly-eye-white-left.svg`
- `assets/jelly-eye-white-right.svg`
- `assets/jelly-pupils.svg`
- `assets/jelly-shadow-new.svg`
