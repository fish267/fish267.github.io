---
author: fish
comments: true
layout: post
title: Tkinter 使用小结
categories: Python
tags: tkinter gui notes
---

###很喜欢PyQt的，但是Tkinter作为Python自带的GUI库，还是不得不了解一下，挺简介方便的，一共包含了15个组件：

1. <code>Button</code> 按钮。类似标签,但提供额外的功能,例如鼠标掠过、按下、释放以及键盘操作/事件

2. <code>Canvas</code> 画布。提供绘图功能(直线、椭圆、多边形、矩形)  ;可以包含图形或位图

3. <code>Checkbutton</code> 选择按钮。一组方框,可以选择其中的任意个(类似 HTML 中的 checkbox)

4. <code>Entry </code> 文本框。单行文字域,用来收集键盘输入(类似 HTML 中的 text)

5. <code>Frame</code>  框架。包含其他组件的纯容器

6. <code>Label</code>  标签。用来显示文字或图片

7. <code>Listbox</code> 列表框。一个选项列表,用户可以从中选择

8. <code>Menu</code>  菜单。点下菜单按钮后弹出的一个选项列表,用户可以从中选择

9. <code>Menubutton</code> 菜单按钮。用来包含菜单的组件(有下拉式、层叠式等等)

	<!--more-->
10. <code>Message</code> 消息框。类似于标签,但可以显示多行文本

11. <code>Radiobutton</code> 单选按钮。一组按钮,其中只有一个可被“按下” (类似 HTML 中的 radio)

12. <code>Scale </code> 进度条。线性“滑块”组件,可设定起始值和结束值,会显示当前位置的精确值

13. <code>Scrollbar</code>  滚动条。对其支持的组件(文本域、画布、列表框、文本框)提供滚动功能

14. <code>Text</code>  文本域。 多行文字区域,可用来收集(或显示)用户输入的文字(类似 HTML 中的 textarea)

15. <code>Toplevel</code>  顶级。类似框架,但提供一个独立的窗口容器。

* * *

写了一点例子，留着查看：

##直接上个图吧，在注释里解释．第一个图：

![](http://bcs.duapp.com/love67/%2Fa.png?sign=MBO:OsG38oCXSPWgEul2SOX0fBg4:C2pQvkvj6d8VMhDBGcbaVyz%2B2sU%3D)
{% highlight python %}   
    #coding:utf-8
    from Tkinter import *
    import sys
    # 创建主窗口
    root = Tk()
    # 设置主窗口的大小，出现的位置
    root.geometry("250x500+400+400")
    # 添加Button, command 绑定行为
    button1 = Button(root, text = 'button1', fg = 'black')
    button2 = Button(root, text = 'quit', fg = 'gray', command = sys.exit)
    button1.pack()
    button2.pack()
    # 添加Label
    label = []
    for i in range(5):
        label.append(Label(root, text = '-' * 80))
    label[0].pack()
    # 添加Canvas,画个图
    canvas = Canvas(bg = 'blue')
    coord = 10, 50, 220, 230
    canvas.create_arc(coord, start = 0, extent = 333, fill = 'yellow')
    canvas.pack()
    label[1].pack()
    # 添加Checkbutton, 就是勾选框
    checkbutton1 = Checkbutton(root, text = 'Python')
    checkbutton2 = Checkbutton(root, text = 'Ruby  ')
    checkbutton3 = Checkbutton(root, text = 'Lisp  ')
    checkbutton1.pack()
    checkbutton2.pack()
    checkbutton3.pack()
    label[2].pack()
    # 添加Entry, 可以输入一行文本
    entry = Entry(root)
    entry.pack()
    label[3].pack()
    # 添加Spinbox
    spinbox = Spinbox(root, from_ = 0, to = 100)
    spinbox.pack()
    label[4].pack()
    # 添加Listbox, 一个列表箱
    listbox = Listbox(root)
    listbox.insert(1, 'C++')
    listbox.insert(2, 'Java')
    listbox.insert(3, 'php')
    listbox.pack()
    mainloop()
{% endhighlight %}
##第二个图：

![](http://bcs.duapp.com/love67/%2FScreenshot-from-2013-12-17-203314.png?sign=MBO:OsG38oCXSPWgEul2SOX0fBg4:GdM0Vjn8kuFpMYtdJmAjvUsr3vg%3D)
{% highlight python %}   
    #coding: utf-8
    from Tkinter import *
    import tkMessageBox
    import sys
    root = Tk()
    root.geometry("200x200+300+300")
    # add tkMessagebox, just a info window
    def action():
        tkMessageBox.showinfo('hello, I am title!', 'hello, Tkinter!!!!')
    # 这个button绑定的是弹出信息窗
    button = Button(root, text = 'responce button', command = action, fg = 'blue')
    # 这个button绑定的行为是推出该GUI
    quitbutton = Button(root, text = 'quitbutton', command = sys.exit)
    quitbutton.pack()
    button.pack()
    mainloop()
{% endhighlight %}
##图三：

![](http://bcs.duapp.com/love67/%2Faa.png?sign=MBO:OsG38oCXSPWgEul2SOX0fBg4:kWfcCX1YXybGqw9HK8gY24hGsXo%3D)
{% highlight python %}   
    #coding: utf-8
    from Tkinter import *
    import tkMessageBox
    import sys
    root = Tk()
    root.geometry("200x200+300+300")
    # 添加Frame
    # Frame框架像是一个容器，负责安放这一堆部件的位置
    # Frame采用矩形区域布局，并提供这些部件的填充
    # 一个框架也可以作为一个基础类，以实现复杂的构件
    frame = Frame(root)
    frame.pack()
    bottomframe = Frame(root)
    bottomframe.pack(side = BOTTOM)
    # 添加text，一个文本组件
    text = Text(bottomframe)
    text.insert(END,'iFrame框架像是一个容器')
    #text.pack()
       
    # 布置几个button
    redbutton = Button(frame, text = 'red', fg = 'red')
    bluebutton = Button(frame, text = 'blue', fg = 'blue')
    yellobutton = Button(frame, text = 'yellow', fg = 'yellow')
    redbutton.pack(side = LEFT)
    bluebutton.pack(side = LEFT)
    yellobutton.pack(side = LEFT)
       
    bottombutton = Button(bottomframe, text = 'buttonButton', fg = 'black', command = sys.exit)
    bottombutton.pack(side = BOTTOM)
    # 分割线
    label2 = Label(bottomframe, text = '-' * 90)
    label2.pack()
       
    # 添加Menubutton
    # menubutton 是个下拉菜单，用户点一下，展开，能选择
    mb = Menubutton(bottomframe, text = '下拉才菜单')
    mb.menu = Menu(mb)
    mb['menu'] = mb.menu
    mb.menu.add_checkbutton(label = 'no1 choice')
    mb.menu.add_checkbutton(label = 'no2 choice')
    mb.pack()
           
    text.pack()
    mainloop()
{% endhighlight %}
