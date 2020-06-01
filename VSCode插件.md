---
title: VSCode插件
date: 2019-11-13 21:39:00
tags: VSCode 插件
category: 其他
---

# 记录以下自己常用的VSCode插件

1. ARM : dan-c-underwood.arm
2. Bracket Pair Colorizer : coenraads.bracket-pair-colorizer
3. C/C++ : ms-vscode.cpptools
4. File Template : ralfzhang.filetemplate
5. Markdown All in One : yzhang.markdown-all-in-one
6. Markdown Extended : jebbs.markdown-extended
7. Markdown TOC : alanwalk.markdown-toc (有bug，https://github.com/AlanWalk/markdown-toc/issues/67)
8. Paste Image : mushan.vscode-paste-image
9. Python : ms-python.python
10. Smalise : loyieking.smalise
11. TabNine : tabnine.tabnine-vscode (代码神器)
12. vscode-icons : vscode-icons-team.vscode-icons
13. YARA : infosec-intern.yara
14. kite : Kite is a plugin for your IDE that uses machine learning to give you useful code completions for Python. Start coding faster today. https://kite.com/
15. C-family Documentation Comments
16. Markdown Shortcuts
17. 

# 改变选中高亮颜色

1. 在设置中搜索 `Workbench：Color Customizations` 如下图

![](VSCode插件/2020-02-14-20-07-45.png)

2. 加上选中的文字内容的配置：
```
    "workbench.colorCustomizations": {
        "editor.selectionBackground": "#f5f113",
        "editor.selectionHighlightBackground": "#f5f113",
    },
```
![](VSCode插件/2020-02-14-20-09-29.png)


