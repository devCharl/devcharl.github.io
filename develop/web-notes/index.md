# HTML / PHP 开发踩坑笔记


记录 HTML / JavaScript DOM 操作和 PHP 混排页面时的踩坑笔记。

## HTML / JavaScript

### `disabled` 属性

`disabled` 是布尔属性，对 `input`、`button`、`select`、`option` 等元素有效。注意属性名是 `disabled`，不是 `disable`。

**标准做法：**

```javascript
const el = document.getElementById("id");

// 禁用
el.disabled = true;

// 取消禁用（推荐）
el.removeAttribute("disabled");
el.disabled = false;
```

**HTML 中的陷阱：**

只要 `disabled` 属性存在，元素就会被禁用，**与属性值无关**：

```html
<!-- 以下写法全部处于禁用状态 -->
<input type="button" disabled="false" value="">
<input type="button" disabled="true" value="">
<input type="button" disabled="123" value="">
<input type="button" disabled=" " value="">
<input type="button" disabled="any word" value="">

<!-- 以下写法才是不禁用 -->
<input type="button" value="">
```

**JS 赋值的行为：**

```javascript
// 以下赋值均会禁用元素
el.disabled = true
el.disabled = {}
el.disabled = "11"

// 以下赋值可取消禁用
el.disabled = false
el.disabled = ""
el.disabled = null
el.disabled = undefined
el.removeAttribute("disabled")
```

### `<span>`

`<span>` 是行内容器，本身不带样式，通常配合 CSS 对局部文字做修饰：

```html
<p>普通文字 <span style="color: red; font-weight: bold;">高亮文字</span> 继续普通文字</p>
```

### `<table>`

默认情况下，`<table>` 中各列 `td` 宽度会随最长内容撑开，长文本容易溢出。

推荐组合样式：

```html
<table style="table-layout: fixed; width: 100%; overflow-wrap: break-word; word-break: break-all;">
```

| 属性 | 说明 |
|------|------|
| `table-layout: fixed` | 列宽由表格宽度和列宽设定决定，不随内容无限撑开 |
| `overflow-wrap: break-word` | 优先在单词边界换行；仍溢出时再断行（`word-wrap` 的旧写法） |
| `word-break: break-all` | 允许在任意字符处断行，适合无空格的长字符串 |

### `<td>`

通过 `text-align` 和 `vertical-align` 控制单元格内容对齐：

```html
<td style="text-align: left; vertical-align: middle;"></td>
```

| 属性 | 作用 |
|------|------|
| `text-align` | 水平对齐（`left` / `center` / `right`） |
| `vertical-align` | 垂直对齐（`top` / `middle` / `bottom`） |

### `<select>`

按选项文本禁用 `select` 中的特定 `option`：

```javascript
const select = document.getElementById("WlanAuthMode_select");
const disabledValues = new Set([
  "Shared",
  "WPA Pre-Shared Key",
  "WPA/WPA2 Pre-Shared Key",
]);

for (const option of select.options) {
  if (disabledValues.has(option.text)) {
    option.disabled = true;
  }
}
```

注意：

- 遍历 `options` 应使用 `for...of` 或索引循环，不要用 `for...in`
- 读取选项文字用 `option.text` 或 `option.textContent`，不要用 `innerText`
- JS 中设置禁应用 `option.disabled = true`，不要赋字符串 `"disabled"`

## PHP

### 未解决问题

在 PHP 与 HTML 混排的页面中，若 JavaScript 代码写在 PHP 输出逻辑之外，或 `<script>` 标签未正确闭合，可能导致后续 HTML 无法渲染，页面表现为「全部空白」。

当时尝试在 PHP 中补一行：

```php
echo '<script language="javascript"></script>';
```

页面才恢复显示，但根因尚未定位。`language` 属性在 HTML5 中已废弃，后续应优先检查：

1. `<script>` 标签是否成对闭合
2. PHP 的 `echo` / `print` 是否打断了 HTML 结构
3. 浏览器控制台是否有 JS 语法错误导致解析中断

### 已解决问题

| 问题 | 解决方法 | 备注 |
|------|----------|------|
| 自定义 JS 函数报错 | 将函数定义放在 `<script>...</script>` 块内 | 不要写在 PHP 字符串拼接的半截位置 |
| 避免重复加载文件 | 使用 `require_once` | 防止函数/常量重复定义 |
| 设置框架组成和属性 | 使用 `frameset` / `frame` | HTML5 已废弃，新项目勿用 |
| 检查变量是否已设置 | 使用 `isset($var)` | 避免未定义变量告警 |
| 忽略错误输出 | 在表达式前加 `@` | 如 `@fopen(...)`，仅用于明确可忽略的场景 |
| 获取数组当前键名 | 使用 `key($array)` | 配合 `next()` / `reset()` 遍历 |
| 将数组指针移到第一个元素 | 使用 `reset($array)` | 返回第一个元素的值 |
| 将数组指针移到下一个元素 | 使用 `next($array)` | 返回下一个元素的值 |
| PHP 短标签输出 | 使用 `<?php ?>` | `<? ?>` 依赖 `short_open_tag`，很多环境默认关闭 |


---

> 作者: [Charles Y](https://github.com/devCharl/devcharl.github.io)  
> URL: https://devcharl.github.io/develop/web-notes/  

