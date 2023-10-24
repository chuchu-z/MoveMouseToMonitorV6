# 利用 AutoHotKey 实现 win10 多显示器（扩展模式）快速切换



## 前言

在日常办公中我们经常会使用到外接显示屏来辅佐我们 提高办公效率

然而多显示屏每次切换屏幕都要用鼠标来移动切换, 对鼠标的依赖非常重且非常麻烦😡

尤其是我左边显示屏是 **IDE** 或者 **shell** 终端, 右边是浏览器

于是乎在知乎上看到一篇这样的文章👇

[请问win10接双显示器（扩展模式）怎么把鼠标游标切换到第二台显示器上？](https://www.zhihu.com/question/50002939/answer/2482798775)



是的, 这看起来能够完美解决我的问题, 于是乎我安装了答主分享的 **MoveMouseToMonitor.exe** 工具

然而这里似乎不够完美, 因为它仅仅实现了鼠标的光标由 **A显示屏** 👉 **B显示屏** 的移动

而实际的聚焦效果仍然保留在原来的 **A显示屏** 上

通俗易懂来讲就是, **鼠标** 和 **键盘** 的聚焦是两码事

这样就导致我的鼠标光标从 **A显示屏** 的 **shell终端** 或者  **IDE** 切换到**B显示屏** 的浏览器后

键盘**输入**时, 实际的**输出**仍然会停留在  **shell终端**  或者  **IDE** 的上

其根本原因为, **键盘**的输出聚焦效果没有完成切换, 而我要的就是把鼠标以及键盘的输入聚焦都切换到另一显示屏

这。。。实现了但没完全实现我想要的效果😂



## 修改

于是我就花了亿点点时间研究了下 **AutoHotKey** 的语法和这个**MoveMouseToMonitor.exe** 程序的源代码文件**MoveMouseToMonitor.ah**

然后再此基础上做了一点点改动, 当执行完 **PerformMove** 函数成功移动鼠标后

创建一个 **ActivateWindowByMousePosition** 函数,  获取当前鼠标位置的窗口 `title`

`WinActivate` 会根据鼠标窗口的`title`来激活目标窗口, 从而真正实现切换并激活窗口, 达到键鼠聚焦同步👍

```autohotkey
PerformMove(MoveMonNum, OffX, OffY)
{
    global MoveX, MoveY
    Gosub, CheckNumMonsChanged
    RestoreDPI := DllCall("SetThreadDpiAwarenessContext", "ptr", -3, "ptr")
    SysGet, Coordinates%MoveMonNum%, Monitor, %MoveMonNum%
    Left := Coordinates%MoveMonNum%Left
    Right := Coordinates%MoveMonNum%Right
    Top := Coordinates%MoveMonNum%Top
    Bottom := Coordinates%MoveMonNum%Bottom
    If (OffX = -1)
        MoveX := Left + (Floor(0.5 * (Right - Left)))
    Else
        MoveX := Left + OffX
    If (OffY = -1)
        MoveY := Top + (Floor(0.5 * (Bottom - Top)))
    Else
        MoveY := Top + OffY
    DllCall("SetCursorPos", "int", MoveX, "int", MoveY)
    Sleep, 10
    DllCall("SetCursorPos", "int", MoveX, "int", MoveY)
    DllCall("SetThreadDpiAwarenessContext", "ptr", RestoreDPI, "ptr")

    Gosub, ActivateWindowByMousePosition

    Return
}

ActivateWindowByMousePosition:
    MouseGetPos, , , id, control
    WinGetTitle, title, ahk_id %id%
    WinActivate, %title%
    Return

```



## AutoHotKey工具

[AutoHotKey 1.1 下载地址](https://www.autohotkey.com/download/ahk-install.exe)

安装完成后打开 **Ahk2Exe.exe** 程序, 把修改好的**MoveMouseToMonitor.ah**文件直接打包即可