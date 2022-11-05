[TOC]

# 后台运行python脚本

# 1 nohup

1、后台运行python代码命令：nohup python3 main.py &
2、nohup 是 no hang up 的缩写，就是不挂起的意思，不断地运行。
3、最后一个 & ，代表该命令在后台执行。
4、命令运行后会有提示，示例： [1] 1111 代表进程 1111 运行中。
5、执行命令ps aux |grep python可以看到python程序，刚刚运行的程序状态为R。
6、关闭程序：kill -9 pid。

# 2 screen

```bash
$ sudo yum install screen

$ screen -S name 

$ screen command #这样新建一个窗口并在窗口中执行command，同样没有名字

例如：screen python ./a.py 新建并执行a.py程序

会话分离

我们在一个窗口运行某个程序之后，想退出登录关闭terminal干点别的事，让程序在后台运行。这时就需要和窗口会话分离,有2种方式：

1)快捷键Ctrl a表示进入命令模式

在当前会话窗口中按Ctrl A +D 快捷键可以实现分离，这时窗口会弹出[detached]的提示，并回到主窗口。

2)screen -d name #远程detach某个session,前提是已经跳出了name窗口

$ screen -ls 
#列出窗口列表

There is a screen on:

2637.count (12/17/2015/10:00:32 AM) (Detached)

恢复回话   screen -r 任务名称

$ screen -r 2637 
#进入2637线程，恢复count会话窗口

这样就能回到count窗口了

杀死会话窗口

如果想关掉一个多余的窗口，有3种方法：

kill -9 threadnum 
例如在上面的2637，kill -9 2637 即可杀死线程，当然就杀死了窗口

使用Ctrl a +k 杀死当前窗口和窗口中运行的程序

使用Ctrl a 然后输入quit命令退出Screen会话。需要注意的是，这样退出会杀死所有窗口并退出其中运行的所有程序

清除死去的窗口

当窗口被杀死后，再用screen -ls 可以看到该窗口后面的(???dead)字样，说明窗口死了，但是仍在占用空间。这时需要清除窗口

$ screen -wipe 
```



