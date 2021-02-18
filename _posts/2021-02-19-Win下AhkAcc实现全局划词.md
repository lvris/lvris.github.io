---
layout: post
title: Win下AhkAcc实现全局划词
date: 2021-02-19 03:00:48
categories: 工具
tags: Ahk Acc
typora-root-url: ..
---

* content
{:toc}
发现了一个Ahk库可以通过调用微软的Api来直接操控一些控件.

所以就写了一个脚本来实现沙拉查词的全局使用(双击取词, 修饰键划词等), 在此分享出来.

不过没有写错误处理, 很有可能会出现Bug, 可以看懂原理后自行编写.

另外, Ahk的基础语法很简单, 所以无论是修改快捷键还是自行编写命令都十分方便.

<!-- more -->

### Acc

#### 简单介绍

[Acc库](https://github.com/sancarn/ACC.AHK)是一个AutoHotKey编写的调用MAAC Api的库, 通过控件的句柄和路径就可以进行一些操作.

在下面的脚本中, 我先使用`GetSaladId()`获取独立窗口的句柄, 这里默认会弹出并显示, 如果你不喜欢的话请注释掉倒数第二行.

之后就可以通过`Salad()`向查词窗口传递参数字符串, 寻找输入栏(input)和搜索按钮(enter)的控件, 修改输入(Value)并点击搜索(Action).

```
global BroswerName := "chrome.exe"
global AccPath := "4.1.1.1.1"   ;chrome
GetSaladId()
{
    global BroswerName
    WinWait, 沙拉查词 ahk_exe %BroswerName%,, 5 ;s
    ControlGet, hWnd, Hwnd
    global SaladId:=hWnd
    MsgBox  % SaladId
    return
}
Salad(search)
{
    global SaladId, AccPath
    input_acc:=Acc_Get("Object", AccPath . ".3", 0, "ahk_id " SaladId)
    input_acc.accValue(1):=search
    enter_acc:=Acc_Get("Object", AccPath . ".4", 0, "ahk_id " SaladId)
    enter_acc.accDoDefaultAction(1)
    return
}
```

同理, 只要寻找到对应的控件路径(如通过[AccSpy](https://link.zhihu.com/?target=https%3A//github.com/serzh82saratov/AhkSpy)), 就能进行大多数操作, 比如说加入生词本按钮的路径是“4.1......6”, 就可以写出如下函数.

Acc_Get参数解释: 返回类型为对象, 控件的路径, 独立窗口的句柄(详情参考[教程](https://www.autohotkey.com/boards/viewtopic.php?f=7&t=31764)).

```
SaladSave()
{
    global SaladId, AccPath
    love_acc:=Acc_Get("Object", AccPath . ".6", 0, "ahk_id " SaladId)
    love_acc.accDoDefaultAction(1)
    return
}
```

#### 获取Path

想要利用Acc, 最关键的就是寻找到控件对应的路径.

这一步有不少轮子, 比如说[AhkSpy](https://github.com/serzh82saratov/AhkSpy).

1. 在打开Spy的Gui之后, 鼠标放在你需要寻找路径的配件上, 使用「Shift+Tab」冻结.

2. 之后在「Control」中寻找「Accessible」栏下的「Get path」, 就可以看到一串数字组成的Path.

对于沙拉查词而言, 不同浏览器的Path不同, 不过工具栏几个控件的编码不变.

比如说输入栏, Chrome中是“4.1.1.1.1.3”, Vivaldi中是“4.1.1.1.2.2.2.1.1.1.1.1.1.1.1.1.1.1.3”.

而搜索键就是「...4」, 爱心是「...6」, 所以我取了它们**前面相同的部分**作为一个全局变量AccPath.

如果要调用我的库的话, 就把AccPath改成对应的path就好(去掉最后一位“3”)

### Ahk

#### 简单介绍 & 划词

Ahk中, 使用「^ ! + #」来代表修饰键「Ctrl Alt Shift Win」, 加上「~」避免屏蔽原按键.

因而可以监听(修饰键)鼠标左键(LButton)松开这个事件, 实现按住修饰键后选中文本即翻译的功能.

理论上讲获取选中文本可以不通过剪贴板, 不过这么写比较方便...

```
; ~#LButton Up::                      ;listen Win+Click(hold)
; ~^LButton Up::                      ;listen Ctrl+Click(hold)
~!LButton Up::Salad(SelectedText())   ;listen Alt+Click(hold)
```

#### Shift多选取词

快捷键很容易自定义, 比如说我下面监听Shift+Click

```
~+LButton::Salad(SelectedText())      ;listen Shift+Click
```

#### 自动加入生词本

再比方说, 如果我想要按住win划词后自动加入生词本, 就连续调用两个函数.

```
~#LButton Up::
    Salad(SelectedText())
    SaladSave()
    return
```

#### 双击取词

Morse()是Ahk论坛上一位网友分享的, 它会监听一个按键, 超时后返回一串模式码(短按0长按1), 这里的”00”就是双击

```
~LButton::                            ;listen Double Click
    if (Morse()="00")
        Salad(SelectedText())
    return
```

#### 不使用剪贴板

获取选中文本貌似也可以通过Acc实现, 但是我还是用了剪贴板实现. (保留原有内容)

不过Acc还可以实现对Dom对象的读取, 这样可以实现Excel里面选中单元格自动获取内容翻译的功能.

读取方法的具体实现可以参考论坛上一位大神写的[教程](https://www.autohotkey.com/boards/viewtopic.php?f=7&t=31764), 这里面好像有操作Dom-Excel的实例.

### 使用方法

1. 安装[Ahk](https://www.autohotkey.com), 最好选择v1的最新版本(Unicode)
2. 下载相关脚本Salad (包含[Acc库](https://github.com/sancarn/ACC.AHK))
3. 修改浏览器相关信息(exe, path等) (Chrome可能不需要改)
4. 修改快捷键, 以符合自己的使用习惯.

#### 不同版本

我提供了一个**内嵌Acc库**的版本(不需要Lib文件夹)和一个**仅包含快捷键**的版本(请修改Lib/Salad.ahk中的相关信息).

当然, 如果你没有自定义需求, 恰好用的还是Chrome, 也可以直接下载exe版本, 不过由于环境不同, 很有可能出现bug.