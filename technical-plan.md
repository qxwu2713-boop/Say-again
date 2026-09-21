# 「再讲一遍」技术方案

## 1. 技术选型

- 单页静态 Web 应用：HTML + CSS + 原生 JavaScript。
- 不依赖框架、构建工具或后端，双击 HTML 文件即可运行。
- 本地持久化：浏览器 `localStorage`。
- 图片：`FileReader.readAsDataURL()`，将本地文件转成 data URL 存储。
- 图标：内联 SVG；避免外部图标依赖。

## 2. 文件结构

```text
zai-jiang-yi-bian/
├── index.html          # 可运行的完整应用
├── requirements.md     # 产品需求
└── technical-plan.md   # 技术方案
```

演示版本可保持单文件；生产化时建议拆分为 `styles.css`、`app.js`、`data.js` 和 `assets/`。

## 3. 数据模型

```js
{
  id: 1710000000000,
  date: '2026.09.08・周二晚',
  text: '完整笔记正文',
  quotes: ['标记为未来回响的句子'],
  image: 'data:image/...;base64,...' // 可选
}
```

数据存储键：`zjyb_notes`。首次读取时若不存在，载入预置笔记数组；每次新增笔记后执行 `localStorage.setItem`。

## 4. 状态设计

```js
let notes;                 // 笔记列表，按时间倒序
let tab = 'home';          // home | record | notes | detail
let detail = null;         // 当前详情笔记 id
let current;               // 当前回响金句及所属笔记
let record = {
  text: '',
  image: null,
  stage: 'write',          // write | mark
  quotes: []
};
```

UI 由 `render()` 根据状态输出对应页面 HTML。小型 Demo 使用字符串模板即可；若迁移到 React / Vue，可将以上状态映射为组件 state。

## 5. 核心逻辑

### 随机回响

1. 使用 `notes.flatMap` 汇总所有 `quotes`。
2. 通过随机下标选择一条；当可选项大于 1 时，避免与当前句子重复。
3. 携带来源笔记信息，以支持「回到那天」。

### 金句拆分与选择

```js
text.split(/(?<=[。！？!?\n])/)
    .map(item => item.trim())
    .filter(Boolean)
```

每一个句子按钮根据 `record.quotes.includes(sentence)` 变更选中样式。完成时允许空数组，表示该笔记仅保存、不参与回响。

### 图片上传

```js
const reader = new FileReader();
reader.onload = () => { record.image = reader.result; };
reader.readAsDataURL(file);
```

注意：data URL 会占用 localStorage 容量。生产版本建议压缩图片、限制图片数量与尺寸，或使用 IndexedDB。

## 6. 字体策略

用户内容优先字体栈：

```css
font-family: "江西拙楷", "Jiangxi Zhuokai", "STKaiti", "KaiTi", "DFKai-SB", cursive;
```

若目标平台未安装「江西拙楷」，应在可用前提下以 WOFF2 文件通过 `@font-face` 内嵌或加载，并保留以上回退。

## 7. 兼容性与验证

- 目标优先：现代 Chromium、Safari、移动端 WebView。
- 使用 `input[type=file]` 的原生图片选择器。
- 验证：首页随机、无金句空态、文本保存、零/多金句、图片预览、刷新后数据恢复、详情跳转和 Tab 切换。
