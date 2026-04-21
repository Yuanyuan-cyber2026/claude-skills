# 📖 法语精读 Skill（Claude 技能）

将任意一篇法语文章，一键生成完整的交互式学习页面——带词汇注释、语法分析和间隔重复闪卡。

> 专为以中文为母语、正在学习法语（B2–C1 水平）的学习者设计。

---

## ✨ 功能概览

将一篇法语文章（链接、文字或 PDF）发给 Claude，这个 skill 会自动生成一个完整的交互学习页面（React JSX），包含以下功能：

| 功能 | 说明 |
|---|---|
| 📄 **高亮精读** | 文章中 B2/C1 词汇自动高亮，点击查看释义 |
| 🇨🇳 **段落中文翻译** | 一键调用 Claude API 翻译每段 |
| 📚 **词汇词典** | 25–40 个 B2/C1 词条，含词性、中文释义、例句 |
| 🔍 **生词查询** | 点击文章中任意陌生词 → Claude 自动查询并加入闪卡 |
| 🧠 **语法分析** | 每篇 5–6 个复杂结构，附中文详解与例句 |
| 🗂️ **闪卡复习** | 3D 翻转闪卡 + 艾宾浩斯遗忘曲线（9 级间隔） |
| ✍️ **仿写练习** | 每段仿写，AI 用中文批改 |
| 💾 **进度保存** | 每篇文章的复习进度独立保存，不会互相干扰 |

---

## 🚀 安装方法

### 第一步 — 下载 skill 文件

从本仓库下载 [`SKILL.md`](./french-reading/SKILL.md)。

### 第二步 — 上传到 Claude.ai

1. 打开 [claude.ai](https://claude.ai)
2. 进入 **Settings（设置）** → **Skills（技能）**
3. 点击 **Upload skill（上传技能）**，选择刚下载的 `SKILL.md`

### 第三步 — 完成

上传后，每当你发给 Claude 一篇法语文章并要求生成学习页面，Claude 会自动调用这个 skill。

---

## 💬 如何触发

将法语文章的文字或者链接粘贴进对话框（或上传 PDF），然后发送：

```
生成这篇文章的交互学习页面。
```

```
Crée une page de lecture interactive pour cet article.
```

Claude 会生成一个 `.jsx` 文件，可以下载后在本地 React/Vite 环境中运行，也可以直接在 Claude 的 artifact 面板中预览。

---

## 🖼️ 功能详解

### 词汇高亮（📖 精读 标签）
- B2 词汇用蓝色高亮，C1 词汇用棕色高亮
- 点击高亮词 → 弹出释义卡片（词性、中文定义、例句、变形）
- 点击**任意其他词** → Claude 实时查询，自动加入闪卡

### 闪卡复习（🗂️ 闪卡 标签）
- 3D 翻转卡片：正面 = 法语词条，背面 = 中文释义 + 例句
- 间隔重复：9 个等级，从 1 分钟到 30 天（艾宾浩斯曲线）
- 按状态筛选：全部 / 待复习 / 新词 / 学习中 / 已掌握
- 键盘快捷键：`空格` = 翻转，`←` = 忘了，`→` = 记住了
- 卡片区域下方显示完整词汇列表，方便随时查阅

### 语法分析
- 每篇文章选取 5–6 个复杂语法结构
- 每条分析包含三部分：
  - 【Structure】语法结构模式标注
  - 【Analyse】中文详细解释 + 法语例句
  - 【Note】用法注意事项与对比
- 法语例句用斜体显示，关键术语加粗

### 仿写练习（AI 批改）
- 每段配有仿写练习区
- 点击「🤖 批改」→ Claude 用中文给出反馈：
  - 🟢 写得好的地方
  - 🔴 错误
  - 🟡 改进建议
  - ✍️ 修改版本

---

## 🏗️ 技术架构

- **输出格式**：单文件 React JSX（所有文章数据以静态常量内嵌）
- **无需在 App 内配置 API Key**：Claude 在生成时已将数据写入文件
- **数据存储**：`window.storage`（Claude artifact 存储），自动回退到 `localStorage`
- **字体**：Crimson Pro（法语正文）· DM Sans（界面）· Noto Sans SC（中文）
- **文章隔离**：每篇文章的词汇和复习进度独立存储，互不影响

---

## 📋 使用要求

- 一个 [Claude.ai](https://claude.ai) 账户（较长文章推荐 Pro 版）
- 如需在本地运行生成的 `.jsx` 文件，需安装 [Node.js](https://nodejs.org) + [Vite](https://vitejs.dev)

```bash
npm create vite@latest my-reading-app -- --template react
cd my-reading-app
# 将 src/App.jsx 替换为下载的文件
npm install && npm run dev
```

---

## 📁 仓库结构

```
french-reading/
├── README.md       ← 英文说明
├── README_ZH.md    ← 中文说明（本文件）
└── SKILL.md        ← Claude skill 指令文件
```

---

## 📄 许可证

MIT 开源协议 — 可自由使用、修改和分享。

---

## 🙏 致谢

本 skill 与 Claude（Anthropic）共同迭代开发，作为个人法语学习工具。

