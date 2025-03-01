# markdown语法([Markdown 官方教程](https://markdown.com.cn/basic-syntax/links.html#code))

[TOC]

## 1、标题语法

#＋空格＋标题

## 2、段落语法

段落从一行开头，不要用空格（spaces）或制表符（ tabs）缩进段落。



用一行空白分割段落

## 3、换行语法

行尾添加“结尾空格”或 HTML 的 `<br>` 标签来实现换行<br>

结尾空格是两个或多个空格  

## 4、强调语法

**粗体**：在单词或短语的前后各添加两个星号（asterisks）或下划线（underscores）

*斜体*：在单词或短语前后添加一个星号（asterisk）或下划线（underscore）

***粗体加斜体***：在单词或短语的前后各添加三个星号或下划线

## 5、引用语法

> 创建块引用，请在段落前添加一个 `>` 符号
>
> 块引用可以包含多个段落。为段落之间的空白行添加一个 `>` 符号
>
> > 块引用可以嵌套。在要嵌套的段落前添加一个 `>>` 符号
> >
> > 块引用可以包含其他 Markdown 格式的元素。并非所有元素都可以使用

## 6、列表语法

1. 有序列表
2. 在每个列表项前添加数字并紧跟一个英文句点。数字不必按数学顺序排列，但是列表应当以数字 1 起始。
3. 1. 列表内可以嵌套
   2. ................

***



- 创建无序列表，请在每个列表项前面添加破折号 (-)、星号 (*) 或加号 (+) 。缩进一个或多个列表项可创建嵌套列表。
- - 无序列表可嵌套
  - 1. 无序列表和有序列表可相互嵌套

---



- 要在保留列表连续性的同时在列表中添加另一种元素，请将该元素缩进四个空格或一个制表符

  > 另一元素

- 连续性

## 7、代码语法

`把代码包裹在反引号中`

转义反引号：`双引号中有双引号再用一对双引号```包裹`

要创建代码块，请将代码块的每一行缩进至少四个空格或一个制表符。

之前和之后的行上使用三个反引号（(`````）或三个波浪号（~~~）,加上代码类型可语法高亮

```json
{
  "firstName": "John",
  "lastName": "Smith",
  "age": 25
}
```

## 8、分割线语法

要创建分隔线，请在单独一行上使用三个或多个星号 (`***`)、破折号 (`---`) 或下划线 (`___`) ，并且不能包含其他内容。

***

---

___

## 9、链接语法

```markdown
[超链接显示名](超链接地址 "超链接title")
```

这是一个链接：[rsr个人博客](http://8.134.176.143:7777 "rsr个人博客")

使用尖括号可以很方便地把URL或者email地址变成可点击的链接：<http://8.134.176.143:7777>  <1004765852@qq.com>

强调链接, 在链接语法前后增加星号。 要将链接表示为代码，请在方括号中添加反引号：

```text
I love supporting the **[EFF](https://eff.org)**.
This is the *[Markdown Guide](https://www.markdownguide.org)*.
See the section on [`code`](#code)
```

## 10、图片语法

```markdown
![图片alt](图片链接 "图片title")
```

**链接图片**：

给图片增加链接，请将图像的Markdown 括在方括号中，然后将链接添加在圆括号中。

[![markdown all in one](https://markdown.com.cn/hero.png "markdown")](https://markdown.com.cn)

```markdown
![markdown all in one](https://markdown.com.cn/hero.png "markdown")](https://markdown.com.cn)
```

## 11、转义字符语法

要显示原本用于格式化 Markdown 文档的字符，请在字符前面添加反斜杠字符 \ 。

特殊：< 和 & 需要写成 < 和 &

## 12、内嵌HTML标签

如需使用 HTML，不需要额外标注这是 HTML 或是 Markdown，只需 HTML 标签添加到 Markdown 文本中即可。

## 13、表格语法

使用三个或多个连字符（`---`）创建每列的标题，并使用管道（`|`）分隔每列。

可用[Markdown Tables Generator](https://www.tablesgenerator.com/markdown_tables)。使用图形界面构建表，然后将生成的Markdown格式的文本复制到文件中。

可以通过在标题行中的连字符的左侧，右侧或两侧添加冒号（`:`），将列中的文本对齐到左侧，右侧或中心。

```markdown
| Syntax      | Description | Test Text     |
| :---        |    :----:   |          ---: |
| Header      | Title       | Here's this   |
| Paragraph   | Text        | And more      |
```

表格可内嵌其他元素，但不能添加标题，块引用，列表，水平规则，图像或HTML标签。

可以使用表格的HTML字符代码（`|`）在表中显示竖线（`|`）字符。

## 14、脚注语法

在方括号（`[^1]`）内添加插入符号和标识符。标识符可以是数字或单词，但不能包含空格或制表符。标识符仅将脚注参考与脚注本身相关联-在输出中，脚注按顺序编号。[^1]

[^1]:脚注1

## 15、删除线

```markdown
在单词前后使用两个波浪号~~
~~这是删除线~~
```

~~世界是平坦的~~

## 16、任务列表

```markdown
- [x] Write the press release
- [ ] Update the website
横杠加空格加[x]可生成已完成任务列表
横杠加空格加[ ]可生成未完成任务列表（中括号中间有空格）
```

- [x] 任务列表

## 17、emoji表情:laughing:

以冒号开头和结尾，并包含表情符号的名称。

```markdown
去露营了！ :tent: 很快回来。
真好笑！ :joy:
```

去露营了！ :tent: 很快回来。
真好笑！ :joy:

## 18、TOC

TOC 全称为 Table of Content，列出全部标题。

```markdown
输入 [TOC] 即可
```

## 19、高级用法

markdown还可做流程图、复杂的公式呈现、序列图、甘特图等等。虽然看起来很有用，但是我认为这些功能与它创立的初衷是违背的，而且做流程图和复杂的公式是有专门的工具，而且十分便捷。所以个人认为，`Markdown`的一些高级用法了解一下即可



---



# 速查表

## [#](https://markdown.com.cn/cheat-sheet.html#基本语法)基本语法

这些是 John Gruber 的原始设计文档中列出的元素。所有 Markdown 应用程序都支持这些元素。

| 元素                                                         | Markdown 语法                                    |
| ------------------------------------------------------------ | ------------------------------------------------ |
| [标题（Heading）](https://markdown.com.cn/basic-syntax/headings.html) | `# H1## H2### H3`                                |
| [粗体（Bold）](https://markdown.com.cn/basic-syntax/bold.html) | `**bold text**`                                  |
| [斜体（Italic）](https://markdown.com.cn/basic-syntax/italic.html) | `*italicized text*`                              |
| [引用块（Blockquote）](https://markdown.com.cn/basic-syntax/blockquotes.html) | `> blockquote`                                   |
| [有序列表（Ordered List）](https://markdown.com.cn/basic-syntax/ordered-lists.html) | `1. First item` `2. Second item` `3. Third item` |
| [无序列表（Unordered List）](https://markdown.com.cn/basic-syntax/unordered-lists.html) | `- First item- Second item- Third item`          |
| [代码（Code）](https://markdown.com.cn/basic-syntax/code.html) | ``code``                                         |
| [分隔线（Horizontal Rule）](https://markdown.com.cn/basic-syntax/horizontal-rules.html) | `---`                                            |
| [链接（Link）](https://markdown.com.cn/basic-syntax/links.html) | `[title](https://www.example.com)`               |
| [图片（Image）](https://markdown.com.cn/basic-syntax/images.html) | `![alt text](image.jpg)`                         |

## [#](https://markdown.com.cn/cheat-sheet.html#扩展语法)扩展语法

这些元素通过添加额外的功能扩展了基本语法。但是，并非所有 Markdown 应用程序都支持这些元素。

| 元素                                                         | Markdown 语法                                                |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [表格（Table）](https://markdown.com.cn/extended-syntax/tables.html) | `| Syntax   | Description || ----------- | ----------- || Header   | Title    || Paragraph  | Text    |` |
| [代码块（Fenced Code Block）](https://markdown.com.cn/extended-syntax/fenced-code-blocks.html) | ````{ "firstName": "John", "lastName": "Smith", "age": 25}```` |
| [脚注（Footnote）](https://markdown.com.cn/extended-syntax/footnotes.html) | Here's a sentence with a footnote. `[^1]` `[^1]`: This is the footnote. |
| [标题编号（Heading ID）](https://markdown.com.cn/extended-syntax/heading-ids.html) | `### My Great Heading {#custom-id}`                          |
| [定义列表（Definition List）](https://markdown.com.cn/extended-syntax/definition-lists.html) | `term: definition`                                           |
| [删除线（Strikethrough）](https://markdown.com.cn/extended-syntax/strikethrough.html) | `~~The world is flat.~~`                                     |
| [任务列表（Task List）](https://markdown.com.cn/extended-syntax/task-lists.html) | `- [x] Write the press release- [ ] Update the website- [ ] Contact the media` |
