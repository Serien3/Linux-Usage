# 调试及性能分析



---

**Resources：**

[调试及性能分析 · the missing semester of your cs education](https://missing-semester-cn.github.io/2020/debugging-profiling/)







---

**代码不能完全按照您的想法运行，它只能完全按照您的写法运行，这是编程界的一条金科玉律。**

A programming language is more than just a means for instructing a computer to perform tasks. The language also serves as a framework within which we organize our ideas about computational processes. Programs serve to communicate those ideas among the members of a programming community. Thus, programs must be written for people to read, and only incidentally for machines to execute.



## 调试代码

“最有效的 debug 工具就是细致的分析，配合恰当位置的打印语句” — Brian Kernighan, *Unix 新手入门*。

调试代码的第一种方法往往是在您发现问题的地方添加一些打印语句，然后不断重复此过程直到您获取了足够的信息并找到问题的根本原因。

另外一个方法是使用日志，而不是临时添加打印语句。日志较普通的打印语句有如下的一些优势：

- 您可以将日志写入文件、socket 或者甚至是发送到远端服务器而不仅仅是标准输出；
- 日志可以支持严重等级（例如 INFO, DEBUG, WARN, ERROR 等），这使您可以根据需要过滤日志；
- 对于新发现的问题，很可能您的日志中已经包含了可以帮助您定位问题的足够的信息。



## 第三方日志系统

如果您正在构建大型软件系统，您很可能会使用到一些依赖，有些依赖会作为程序单独运行。如 Web 服务器、数据库或消息代理都是此类常见的第三方依赖。

和这些系统交互的时候，阅读它们的日志是非常必要的，因为仅靠客户端侧的错误信息可能并不足以定位问题。

幸运的是，大多数的程序都会将日志保存在您的系统中的某个地方。对于 UNIX 系统来说，程序的日志通常存放在 `/var/log`。例如， [NGINX](https://www.nginx.com/) web 服务器就将其日志存放于 `/var/log/nginx`。

目前，系统开始使用 **system log**，您所有的日志都会保存在这里。大多数（但不是全部的）Linux 系统都会使用 `systemd`，这是一个系统守护进程，它会控制您系统中的很多东西，例如哪些服务应该启动并运行。`systemd` 会将日志以某种特殊格式存放于 `/var/log/journal`，您可以使用 [`journalctl`](http://man7.org/linux/man-pages/man1/journalctl.1.html) 命令显示这些消息。

类似地，在 macOS 系统中是 `/var/log/system.log`，但是有更多的工具会使用系统日志，它的内容可以使用 [`log show`](https://www.manpagez.com/man/1/log/) 显示。

对于大多数的 UNIX 系统，您也可以使用 [`dmesg`](http://man7.org/linux/man-pages/man1/dmesg.1.html) 命令来读取内核的日志。

如果您希望将日志加入到系统日志中，您可以使用 [`logger`](http://man7.org/linux/man-pages/man1/logger.1.html) 这个 shell 程序。下面这个例子显示了如何使用 `logger` 并且如何找到能够将其存入系统日志的条目。

不仅如此，大多数的编程语言都支持向系统日志中写日志。

```bash
logger "Hello Logs"
# On macOS
log show --last 1m | grep Hello
# On Linux
journalctl --since "1m ago" | grep Hello
```

正如我们在数据整理那节课上看到的那样，日志的内容可以非常的多，我们需要对其进行处理和过滤才能得到我们想要的信息。

如果您发现您需要对 `journalctl` 和 `log show` 的结果进行大量的过滤，那么此时可以考虑使用它们自带的选项对其结果先过滤一遍再输出。还有一些像 [`lnav`](http://lnav.org/) 这样的工具，它为日志文件提供了更好的展现和浏览方式。



## 调试器

当通过打印已经不能满足您的调试需求时，您应该使用调试器。

调试器是一种可以允许我们和正在执行的程序进行交互的程序，它可以做到：

- 当到达某一行时将程序暂停；
- 一次一条指令地逐步执行程序；
- 程序崩溃后查看变量的值；
- 满足特定条件时暂停程序；
- 其他高级功能。

我们可以理解为，调试器能够让我们以状态机的视角去逐步观察一个程序的运行，来查找出不符合我们预期的状态映射！

