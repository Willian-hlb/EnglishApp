# TechWords — 技术英语词汇学习 PWA

> 面向技术人员的离线英语词汇学习工具，覆盖 AI、GitHub、Linux、Docker、编程等领域。

当前版本：**v1.1.0**

完整版本变化、测试和回滚记录见：[CHANGELOG.md](CHANGELOG.md)。

---

## 目录

1. [项目概述](#1-项目概述)
2. [技术架构](#2-技术架构)
3. [文件结构](#3-文件结构)
4. [数据架构](#4-数据架构)
5. [功能模块说明](#5-功能模块说明)
6. [如何修改单词数据](#6-如何修改单词数据)
7. [如何本地测试](#7-如何本地测试)
8. [如何部署到手机](#8-如何部署到手机)
9. [关键约束与注意事项](#9-关键约束与注意事项)
10. [未来改进方向](#10-未来改进方向)
11. [版本记录](#版本记录)

---

## 1. 项目概述

**用途**：用户在使用 Claude、GitHub、Linux 等英文技术工具时遇到不认识的单词，可以存入此 App 学习，支持间隔重复记忆法。

**目标平台**：iPhone（Safari + 添加到主屏幕），也可在任何现代浏览器中使用。

**核心特性**：
- 完全离线运行（PWA + Service Worker）
- 497 个去重后的技术词汇与 AI 行业概念（数量不设上限）
- 标准释义 + 大白话 + 技术场景 + 科普要点 + 常见搭配 + 易混点
- 每日固定新词、首次认识、自动混合训练、独立到期复习
- 英选中、中选英、拼写/短语排序、听音选词、例句填空
- 错题记录、训练正确率和错题优先复习
- 间隔重复复习算法（Spaced Repetition）
- 数据 100% 存储在用户设备本地，不上传任何服务器
- 单文件架构，无需构建工具，无框架依赖

**当前部署地址**：`https://animated-shortbread-e75055.netlify.app`（Netlify 免费托管）

---

## 2. 技术架构

```
┌─────────────────────────────────────────────────┐
│                 iPhone Safari                   │
│  ┌──────────────────────────────────────────┐   │
│  │          TechWords PWA (离线)             │   │
│  │                                          │   │
│  │  index.html ──── 全部 UI + 逻辑 (单文件) │   │
│  │  words-data.js ─ 497个词汇与AI科普数据   │   │
│  │  sw.js ──────── Service Worker (离线缓存)│   │
│  │  manifest.json ─ PWA 安装配置            │   │
│  └──────────────────────────────────────────┘   │
│                    ↕ 读写                        │
│  ┌──────────────────────────────────────────┐   │
│  │         iPhone localStorage              │   │
│  │  tw_words    ─ 所有词汇 + 学习状态       │   │
│  │  tw_settings ─ 用户设置（每日目标等）    │   │
│  │  tw_stats    ─ 学习统计数据              │   │
│  │  tw_meta     ─ 数据结构/App版本信息      │   │
│  └──────────────────────────────────────────┘   │
└─────────────────────────────────────────────────┘
```

**技术选型**：
- 纯原生 HTML/CSS/JS，无任何框架或构建工具
- PWA：`manifest.json` + `sw.js` 实现可安装 + 离线
- 数据持久化：`localStorage`（浏览器本地存储）
- 间隔复习算法：复习间隔 `[3, 7, 14, 30, 90]` 天，标记"认识"后按此序列推进

---

## 3. 文件结构

```
EnglishApp/
│
├── index.html            # ★ 核心文件，整个 App 的 UI + 全部业务逻辑
│                         #   约 1180 行，内含 CSS 和 JavaScript
│
├── words-data.js         # ★ 497个词汇与AI行业概念
│                         #   向 window.INITIAL_WORDS 数组赋值
│                         #   保留原主题分批注释，ID不要求连续
│
├── words-data.backup.js  # v1.0.0 原始500词备份，用于紧急回滚
│
├── sw.js                 # Service Worker：离线缓存策略
│                         #   网络优先、离线回退；缓存页面、词库和图标
│
├── manifest.json         # PWA 配置：App名称、图标、显示模式等
│                         #   display: standalone = 全屏无地址栏
│
├── icon.svg              # App 图标（矢量格式，所有平台通用）
├── icon-192.png          # iOS 主屏幕图标（192×192，必须 PNG）
├── icon-512.png          # PWA 安装图标（512×512）
│
├── CHANGELOG.md          # 正式版本变化、测试与回滚记录
│
└── README.md             # 本说明文件
```

**父目录（`claude code/`）：**
```
claude code/
├── EnglishApp/              # App 本体（见上）
├── 启动手机访问.bat          # 双击启动局域网服务器，手机同 WiFi 可访问
└── 部署到互联网（永久）.bat  # 双击通过 Netlify CLI 重新部署（更新内容用）
```

---

## 4. 数据架构

### 4.1 words-data.js — 预置词汇

```javascript
window.INITIAL_WORDS = [
  {
    id: 'w001',              // 唯一ID，格式固定为 w + 3位数字
    word: 'hallucination',   // 英文单词
    phonetic: '/həˌluːsɪˈneɪʃn/', // IPA 国际音标（美式）
    zhMeaning: '幻觉（AI错误生成）',  // 中文释义
    enMeaning: 'When an AI model confidently generates false information.', // 英文释义
    example: 'The chatbot\'s hallucination led it to cite a non-existent paper.', // 英文例句
    exampleZh: '聊天机器人产生幻觉，引用了一篇不存在的论文。', // 例句中文翻译
    source: 'AI',            // 来源分类（AI/GitHub/Linux/Docker/编程/开发工具/DevOps/API/产品）
    tags: ['AI', 'LLM'],     // 标签数组（可多个）
    notes: '',               // 备注（可为空字符串）
    status: 'unknown',       // 学习状态：'unknown'|'fuzzy'|'known'
    reviewCount: 0,          // 复习次数
    starred: false           // 是否收藏
  },
  // ... v1.1.0 共497个（去重后并新增AI行业词汇）
];
```

**重要**：运行时学习字段不在 `words-data.js` 中定义，由 `index.html` 的 `initializeData()` 在首次加载或数据升级时补充。

### 4.2 localStorage 数据结构

| Key | 类型 | 内容 |
|-----|------|------|
| `tw_words` | JSON Array | 所有词汇（含学习状态、复习时间等运行时字段） |
| `tw_settings` | JSON Object | 用户设置：`dailyNew`、`dailyReview`、显示选项 |
| `tw_stats` | JSON Object | 统计：`streak`、`lastStudyDate`、`history` |
| `tw_meta` | JSON Object | `schemaVersion`、`appVersion`、迁移时间 |
| `tw_daily_session` | JSON Object | 当日固定学习/复习词、阶段、任务、进度和完成状态 |

### 4.3 词汇在 localStorage 中的完整结构

词条还可以包含以下可选内容字段：

```javascript
plainMeaning: '大白话解释',
contextMeaning: '技术场景解释',
knowledge: ['科普要点1', '科普要点2'],
collocations: ['常见搭配'],
confusions: '易混点'
```

首次加载后，`tw_words` 中每个词条会增加以下运行时字段：

```javascript
{
  // ... words-data.js 中的所有字段 +
  createdAt: 1750000000000,   // 时间戳（毫秒），首次加载时设置
  nextReview: 1750000000000,  // 下次复习时间戳
  attemptCount: 0,            // 已作答次数，区分新词和复习词
  lastReviewedAt: null        // 最近一次作答时间
}
```

v1.1.0 会自动迁移旧版数据：合并确认重复的预置ID，保留收藏和学习状态，并自动补入新版AI词条与缺失科普字段。用户自行添加的词条不会参与预置ID合并。

### 4.4 间隔复习算法

```
完成首次新词学习 → 次日进入第一次复习

历史复习全部答对时，nextReview 按以下序列推进（天数）：
  +3天 → +7天 → +14天 → +30天 → +90天

历史复习中有错误但随后答对 → 24小时后再次复习
历史复习中未答对 → 6小时后再次复习
```

首次学习和历史复习分开管理。错题会先追加到当前训练队列后部，正式完成后再根据整轮表现安排下一次复习。

---

## 5. 功能模块说明

`index.html` 中各页面对应的渲染函数（全部在底部 `<script>` 块中）：

| 页面 | 导航栏 | 渲染函数 | 说明 |
|------|--------|----------|------|
| 首页 | 首页 | `homeRender()` | 今日计划、统计卡片、最近添加 |
| 学习页 | 学习 | `learnInit()` / `learnShow()` | 首次认识新词，然后自动进入混合训练 |
| 复习页 | 首页“开始复习” | 与学习页共用状态机 | 仅训练历史到期词，错题自动重入队 |
| 添加单词 | 添加 | `addInit()` | 表单填写、来源/标签选择 |
| 词库 | 词库 | `libraryRender()` | 搜索、筛选、单词列表、点击查看详情 |
| 统计 | 统计 | `statsRender()` | 掌握进度、7天学习图、来源/标签分布 |

**页面路由**：通过 `navigate(page)` 函数切换，App 为单页应用（SPA），无 URL 跳转。

**数据库操作**：通过 `DB` 对象统一操作 localStorage：
```javascript
DB.words.get()           // 读取所有词汇
DB.words.set(array)      // 覆盖全部词汇
DB.words.add(word)       // 新增一个词汇
DB.words.update(id, obj) // 更新指定词汇的部分字段
DB.words.remove(id)      // 删除指定词汇
```

---

## 6. 如何修改单词数据

### 新增词汇（在 words-data.js 末尾追加）

在 `words-data.js` 文件最后一行 `];` 之前插入：

```javascript
{id:'w501',word:'新单词',phonetic:'/音标/',zhMeaning:'中文意思',enMeaning:'English definition.',example:'Example sentence.',exampleZh:'例句中文翻译。',source:'AI',tags:['AI'],notes:'',status:'unknown',reviewCount:0,starred:false},
```

注意：
- `id` 必须唯一。现有ID因去重存在空号，不得复用已经使用或迁移过的ID
- `status` 固定填 `'unknown'`，`reviewCount` 固定填 `0`，`starred` 固定填 `false`
- 不要填运行时字段：`createdAt`、`nextReview`、`attemptCount`、`lastReviewedAt`

### 修改现有词汇

直接在 `words-data.js` 中找到对应 `id` 的词条修改字段即可。

### 修改后如何生效

**新用户**：自动生效（首次加载时读取 words-data.js）

**已有数据的设备**：v1.1.0 会自动迁移学习状态，但不会强制覆盖用户手工编辑过的词义。后续新增预置词的批量同步需要继续通过数据版本迁移实现，不能要求用户清空学习记录。

### 修改词汇类别/来源

`source` 字段的可选值（在 index.html 顶部 `SOURCES` 常量中定义）：
```javascript
['AI','Claude','ChatGPT','GitHub','Linux','Docker','DevOps','API','开发','编程','开发工具','技术文档','产品','VPN','投标文件','其他']
```

如需新增来源类别，同步修改 index.html 中的 `SOURCES` 数组。

---

## 7. 如何本地测试

### 方法一：双击脚本（推荐）

双击父目录下的 `启动手机访问.bat`，自动启动服务器并显示访问地址。

### 方法二：命令行启动

```bash
# 在 claude code/ 目录下执行
npx serve -l 3456 EnglishApp
# 然后浏览器打开 http://localhost:3456
```

### 数据重置（测试时常用）

在浏览器控制台（F12）执行：

```javascript
// 清空所有数据，重新加载当前预置词库
localStorage.removeItem('tw_words');
localStorage.removeItem('tw_settings');
localStorage.removeItem('tw_stats');
localStorage.removeItem('tw_meta');
location.reload();
```

### 验证词汇数据完整性

在 `EnglishApp/` 目录下执行：

```bash
node -e "
let code = require('fs').readFileSync('words-data.js','utf8').replace('window.INITIAL_WORDS','global.INITIAL_WORDS');
eval(code);
const w = global.INITIAL_WORDS;
console.log('总词数:', w.length);
const ids = w.map(x=>x.id);
console.log('重复ID:', ids.filter((id,i)=>ids.indexOf(id)!==i));
console.log('缺少必填字段:', w.filter(x=>!x.word||!x.phonetic||!x.zhMeaning).map(x=>x.id));
"
```

### 测试清单

- [ ] 首页：全新数据的总词汇显示 497
- [ ] 学习页：单词卡正常显示，三个状态按钮可点击，点击后推进下一张
- [ ] 当天首次开始学习后固定选择每日新词，退出重进顺序不变
- [ ] 首次认识阶段只有上一个/下一个，不显示熟练度按钮
- [ ] 学完后自动进入混合训练，用户不能手动切换题型
- [ ] 答错后词条追加到本轮后部再次出现
- [ ] 答对或答错后都自动朗读正确英文
- [ ] 学习完成后按钮禁用，当天不能重新开始正式学习
- [ ] 到期历史词只出现在“开始复习”，完成后复习按钮禁用
- [ ] 旧版500词数据：自动迁移为497个新版词条（另加用户自定义词），学习记录保留
- [ ] 搜索 LLM、GLM、MoE、harness、VRAM、INT4 等词条可查看科普
- [ ] 六种训练题型由系统自动穿插，均可作答、显示反馈并推进
- [ ] 统计页显示训练正确率和错题词数
- [ ] 词库页：搜索功能、筛选器（全部/已掌握/模糊/不认识）
- [ ] 统计页：来源分布、标签分布数据正确
- [ ] 添加单词：填写表单后保存，词库页能找到新词
- [ ] Service Worker：第一次加载后断网，刷新仍可用

---

## 8. 如何部署到手机

### 重新部署（更新了词汇数据后）

双击父目录下的 `部署到互联网（永久）.bat`，或在命令行执行：

```bash
npx netlify-cli deploy --dir=EnglishApp --prod
```

### iPhone 安装步骤

1. Safari 打开 `https://animated-shortbread-e75055.netlify.app`
2. 等页面完全加载（首次需要 3~5 秒，之后加载缓存）
3. 点底部「分享」按钮（方块+向上箭头图标）
4. 点「添加到主屏幕」→「添加」
5. 桌面出现图标，以后从图标打开，完全离线可用

### Service Worker 缓存更新

v1.1.0 使用网络优先、离线回退策略。在线启动时会请求最新文件，断网时使用缓存。

每次正式发布仍必须同步修改 `sw.js` 的缓存版本号：

```javascript
// sw.js 第1行
const CACHE_NAME = 'techwords-v1.1.0';
```

---

## 9. 关键约束与注意事项

### ⚠️ 不能做的事

1. **不能在 words-data.js 中添加运行时学习字段**
   `createdAt`、`nextReview`、`attemptCount`、`lastReviewedAt` 由 `initializeData()` 统一补充和迁移。

2. **不能把 `words-data.js` 改为 ES Module（`export default`）**
   必须保持 `window.INITIAL_WORDS = [...]` 的全局变量写法，因为 sw.js 缓存策略不支持动态模块。

3. **不能用 `var`/`let`/`const` 代替 `window.INITIAL_WORDS`**
   因为主脚本通过 `window.INITIAL_WORDS` 读取，必须挂在全局对象上。

4. **不能修改词条的 `id` 格式**
   已有用户数据中的词是通过 `id` 关联的，修改 id 会导致数据错乱。

### ✅ 加载顺序（重要）

```html
<!-- index.html 底部，顺序不能调换 -->
<script src="words-data.js"></script>   <!-- 先加载词汇数据 -->
<script>
  // 主逻辑，最后执行
  // initializeData() 在此处调用，依赖 window.INITIAL_WORDS 已存在
</script>
```

### localStorage 容量

当前497词仍使用 localStorage。词汇和科普字段持续增加后，需要监控实际存储占用；接近浏览器配额前应迁移到 IndexedDB。

### iOS Safari 特殊行为

- Service Worker 必须 HTTPS 才能注册（本地测试用 `localhost`，生产用 Netlify）
- `apple-touch-icon` 必须是 PNG 格式（不支持 SVG），已配置 `icon-192.png`
- Safari 不支持 `beforeinstallprompt` 事件，"添加到主屏幕"只能手动操作

---

## 10. 未来改进方向

| 功能 | 说明 | 优先级 |
|------|------|--------|
| 高级训练 | 完整键盘听写、易混词专项、收藏词专项 | 中 |
| 更多词条科普 | 逐步为普通技术词补齐大白话和易混点 | 中 |
| 单词本分组 | 按项目/场景建立不同词汇集（如"工作用词"/"面试词汇"）| 中 |
| 词汇搜索跳转 | 在学习页长按单词直接搜索例句或跳转词典 | 中 |
| 深色/浅色主题切换 | 当前为深色主题，可加入浅色模式 | 低 |

---

## 版本记录

详细记录见：[CHANGELOG.md](CHANGELOG.md)。

### v1.1.0（2026-06-22）

- 每日新词固定，重复进入不再换词
- 学习和复习拆分为两个独立入口
- 首次学习只浏览解释并点击上一个/下一个
- 系统自动编排训练题型，取消手动切换
- 答对答错都会朗读正确英文单词
- 错题在本轮后部再次出现
- 学习和复习完成后入口锁定
- 新增 `tw_daily_session` 保存当日进度
- 修复词库搜索，精确词条优先并增加结果数和清空按钮

### v1.0.1（2026-06-22）

- 修复新词选择“模糊”后退出学习队列的问题
- 修复“不认识”词未到期可能再次进入新词队列的问题
- 日期统计改为设备本地日期，避免中国时区凌晨错日
- 增加 `attemptCount`、`lastReviewedAt` 和 `tw_meta`
- 为旧版数据增加兼容迁移，保留学习状态和收藏
- 删除37条确认重复的预置记录
- 保留 `pipeline`、`hook`、`environment` 三组不同语境词条
- 统一添加页来源与标签选项
- Service Worker 升级为网络优先、离线回退
- 新增 LLM、GLM、MoE、VRAM、FP16、INT8、INT4、Agent 等34个AI行业词条
- 为 parameter、context window、quantization、harness 等重点词加入通俗释义和科普
- 预置词库最终为497条，并可自动同步到旧设备
- 加入英选中、中选英、拼写/短语排序、听音选词、例句填空训练
- 加入错题记录、训练正确率和错题优先复习

### AI术语核验参考

- GLM 原始论文：<https://arxiv.org/abs/2103.10360>
- Mixtral MoE 总参数与激活参数说明：<https://arxiv.org/abs/2401.04088>
- Hugging Face Transformers 量化文档：<https://huggingface.co/docs/transformers/quantization/bitsandbytes>
- Anthropic《Building effective agents》：<https://www.anthropic.com/research/building-effective-agents>

---

## 维护联系

此项目由 Claude（Anthropic）与用户协作开发，2026年6月。  
词汇数据均来源于真实技术文档、GitHub、Linux 手册等，禁止编造。  
如有 AI 后续维护，请阅读本文档后再修改代码，避免破坏数据格式。
