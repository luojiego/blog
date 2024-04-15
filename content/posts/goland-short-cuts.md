---
title: goland 25 大特色编辑功能
date: 2020-04-19 10:13:32
tags: 
- goland
---
# 写在前面
**使用了Goland两年了，好多功能都不会用，特意总结一下，加强自己的记忆**

[GoLand IDE 的 25 大特色编辑功能 – 第 1 部分](https://blog.jetbrains.com/cn/2019/11/goland-ide-p1/)
[GoLand IDE 的 25 大特色编辑功能 – 第 2 部分](https://blog.jetbrains.com/cn/2019/11/goland-ide-p2/)
[GoLand IDE 的 25 大特色编辑功能 – 第 3 部分](https://blog.jetbrains.com/cn/2020/02/goland-ide-3/)
<!--more-->
# Mac 快捷健简写
**真是惭愧，Mac 买来都 5 年了，快捷键简写都搞不清楚，每次看到了，随便按一按，不能工作就放弃，我真是个小天才**

| 简写 | 对应 |
| --- | --- |
| ⌘ | Command |
| ⌃ | Ctrl |
| ⌥ | Alt |
| ⇧ | Shift |
| ⇪ | Caps lock | 

# 快捷键及说明
**按照我理解的优先级来记录，并不一定有25项，不熟悉快捷键的小伙伴，可以进入官方博客，有详细的动态图说明，写的非常详细**

| 功能 | Windows/Linux | Mac | 备注 |
| --- | --- | --- | --- |
| 最近的位置 | Ctrl+Shift+E | Command+Shift+E | 最厉害的功能，展示最近查看和修改过的位置，再次输入，展示修改过的内容
| 跳到导航栏 | Alt+Home | Command+↑ | 可以在当前文件所在位置，从文件目录对应的位置寻找其它文件，可以搜索
| 从剪贴板历史粘贴 | Ctrl+Shift+V | Command+Shift+V | 特别实用的一个功能 |
| 本地历史记录 | Shift+Shift | Shift+Shift | 输入 Local History, 方便查看选中的区域或者文件的历史个性记录，特别是在没有版本控制时，非常有用 |
| 隐藏所有工具窗口 | Ctrl+Shift+F12 | Command+Shift+F12 | 隐藏 IDE 中的所有工具窗口，快速进入编辑窗口 |
| 随处搜索 | Shift+Shift | Shift+Shift | 搜索任何东西，甚至可以从搜索结果中切换设置 |
| 快速输入 | 无 | 无 | 在列表中输入任何东西并且可以筛选结果。随后可以使用方向键在列表中移动，或者按下 Esc 取消筛选器 |
| 实现接口 | Ctrl+I | Ctrl+I | 按键之后输入需要实现的接口就可以轻松来实现了 |
| 结构体标记 | 无 | 无 | 在字段后输入json或者xml,按table会自动填充，也可自定义.进入 Settings/Preferences \| Editor \| Live Templates，然后选择 Go Struct Tags，即可添加自己的结构字段标记。
| 生成测试 | Ctrl+Shift+T | Command+Shift+T | 快速生成测试，可以根据函数，也可以根据文件生成测试，同时该快捷健也可以在测试文件和原文件之间切换
| 展开选择 | Ctrl+W | Option+↑ | 不断扩展选择区域
| 多重选择 | Ctrl+J | Ctrl+G | 选择一个符号，按下快捷键，会寻找下一个符合条件的符号，可以修改多个符号 
| 导航至文件 | Ctrl+Shift+N | Command+Shift+O | 快速定位到文件，相比Shift+Shift (默认在All标签)，指定到了文件标签
| 多个编辑文件切换 | Ctrl+Table | Ctrl+Table | 也可按住 Ctrl, 再按 Table, 则可在多个最近编辑过的文件之间切换
| 最近的文件 | Ctrl+E | Ctrl+E | 展示最近打开的文件窗口，快速选择，还能输入来搜索
| 文件结构窗口 | Ctrl+F12 | Command+F12 | 第一次输入快捷键，展示文件结构，第二次输入，将显示定义在当前文件所属包中的所有元素
| 在特定工具窗口中选择当前选定的文件 | Alt+F1 | Alt+F1 | 试试吧，和我的使用习惯不太一样
| 类型层次结构 | Ctrl+H | Ctrl+H | 选中类型，键入快捷键，看看效果吧
| 调用层次结构 | Ctrl+Alt+H | Ctrl+Alt+H | 顾名思义，查看方法在那些地方被调用了，可以尝试一下标准库的函数
| 显示引用 | Alt+F7 | Alt+F7 | 展示引用的地方路径 |

# 欢迎指正
若您在阅读时，发现了错误，请在评论区写下来，我会尽快指正，避免影响其他小伙伴阅读。

