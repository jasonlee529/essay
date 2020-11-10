---
title: VScode markdown插件收集
date: 2020-11-06 08:45:57
categories:
- tools
- markdown
tags:
- tools
- markdown
---
# 1. Markdown all in one[链接](https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one)
全功能集，下载人数最多的一款，主要用到的格式化功能。
# 2. Markdown Preview Enhanced[链接](https://marketplace.visualstudio.com/items?itemName=shd101wyy.markdown-preview-enhanced)
* 自动编辑器及预览滑动同步
* 导入外部文件
* Code Chunk
* Pandoc
* Prince
* 电子书
* 幻灯片
* 可扩展性
* LaTeX 数学
* 导出 PDF, PNG, 以及 JPEG 凭借 Puppeteer
* 导出漂亮的 HTML（移动端支持）
* 编译到 GitHub Flavored Markdown
* 自定义预览 CSS
* TOC 生成
* 流程图 / 时序图 以及各种其他种类的图形
* 嵌入 LaTeX, 渲染 TikZ, Chemfig 等图形
* Task List (Github Flavored)
* 图片助手
* 脚注
* Front Matter
# 3. Markdown Shortcuts[链接](https://marketplace.visualstudio.com/items?itemName=mdickin.markdown-shortcuts)
快捷键合集，支持右键完成各种格式调整操作：加粗、斜体、表格等等
| Name                            | Description                                      | Default key binding |
| ------------------------------- | ------------------------------------------------ | ------------------- |
| md-shortcut.showCommandPalette  | Display all commands                             | ctrl+M ctrl+M       |
| md-shortcut.toggleBold          | Make \*\*bold\*\*                                | ctrl+B              |
| md-shortcut.toggleItalic        | Make \_italic\_                                  | ctrl+I              |
| md-shortcut.toggleStrikethrough | Make \~\~strikethrough\~\~                       |                     |
| md-shortcut.toggleLink          | Make [a hyperlink]\(www.example.org)             | ctrl+L              |
| md-shortcut.toggleImage         | Make an image ![]\(image_url.png)                | ctrl+shift+L        |
| md-shortcut.toggleCodeBlock     | Make \`\`\`a code block\`\`\`                    | ctrl+M ctrl+C       |
| md-shortcut.toggleInlineCode    | Make \`inline code\`                             | ctrl+M ctrl+I       |
| md-shortcut.toggleBullets       | Make * bullet point                              | ctrl+M ctrl+B       |
| md-shortcut.toggleNumbers       | Make 1. numbered list                            | ctrl+M ctrl+1       |
| md-shortcut.toggleCheckboxes    | Make - [ ] check list (Github flavored markdown) | ctrl+M ctrl+X       |
| md-shortcut.toggleTitleH1       | Toggle # H1 title                                |                     |
| md-shortcut.toggleTitleH2       | Toggle ## H2 title                               |                     |
| md-shortcut.toggleTitleH3       | Toggle ### H3 title                              |                     |
| md-shortcut.toggleTitleH4       | Toggle #### H4 title                             |                     |
| md-shortcut.toggleTitleH5       | Toggle ##### H5 title                            |                     |
| md-shortcut.toggleTitleH6       | Toggle ###### H6 title                           |                     |
| md-shortcut.addTable            | Add Tabular values                               |                     |
| md-shortcut.addTableWithHeader  | Add Tabular values with header                   |                     |
# 4. markdown-toc[链接](https://marketplace.visualstudio.com/items?itemName=AlanWalk.markdown-toc)
非常好用的目录生成工具，
* Insert header number sections. 插入目录段落序号
* Auto active plugin on markdown。 
* Insert anchor for header ```<a id="markdown-header" name="header"></a>``` 插入锚到文件头
* Linking via anchor tags # A 1 → #a-1  自动重拍段落编号
* Depth control[1-6] with depthFrom:1 and depthTo:6 段落编号支持6级
* Enable or disable links with withLinks:true 
* Refresh list on save with updateOnSave:true
* Use ordered list (1. ..., 2. ...) with orderedList:true
