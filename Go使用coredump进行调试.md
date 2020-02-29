# 使用coredump来调试go程序的内存崩溃等问题

## 注意

+ 由于部署在的linux平台，所以本次实践的平台为linux
+ 笔记仅记录了如何dump出`.core`文件，具体的debug过程后续添加

## 以下是具体步骤和遇到的坑

### 设置coredump的路径

coredump在程序奔溃时dump出来的文件的路径需要自己设置，配置文件在`/proc/sys/kernel/core_pattern`,要使用coredump首先需要配置这个路径，修改方法如下：  
`echo '/tmp/core_%e.%p' | sudo tee /proc/sys/kernel/core_pattern`

1. `/tmp/core_%e_%p`为dump出来的文件的路径和文件格式，其中%p为pid,%e为二进制名称，举个例子：如果可执行文件`hello`，pid为2333，那么dump出来的文件路径为`/tmp/`，文件名为`core_hello.2333`  
2. `/proc/sys/kernel/core_pattern`这个文件的坑，这个文件属于内核虚拟文件，所以无法用vim等编辑器进行编辑保存，如果要修改只能通过`echo`将输出重定向到这个文件，但是使用`>`进行重定向都没有管理员权限，所以也可以先拿到管理员权限然后使用：  
`echo '/tmp/core_%e.%p' > /proc/sys/kernel/core_pattern`

### 设置coredump的文件大小

coredump是把程序崩溃一瞬间的内存数据都dump到core文件，所以如果内存很大会导致dump出来的文件会很大  
  `ulimit -c unlimited`  
注意：这样配置只会在当前会话有效果，重新打开一个会话有需要重新设置，如果为了省事可以配置到环境变量里面。

### 设置GOTRACEBACK环境变量

这一步是Go独有的，如果C++要配置前两步就能够dump出来文件，但是Go语言不行，还需要配置一个环境变量  
`export GOTRACEBACK=crash`  
这个环境变量其实不止`crash`一个选项，选项列表如下：

+ GOTRACEBACK=0 只输出panic异常信息。
+ GOTRACEBACK=1 此为go的默认设置值， 输出所有goroutine的stack traces, 除去与go runtime相关的stack frames.
+ GOTRACEBACK=2 在GOTRACEBACK=1的基础上， 还输出与go runtime相关的stack frames,从而了解哪些goroutines是由go runtime启动运行的。
+ GOTRACEBACK=crash, 在GOTRACEBACK=2的基础上，go runtime触发进程segfault错误，从而生成core dump, 当然要操作系统允许的情况下， 而不是调用os.Exit。

### 小结

感觉这玩意儿其实有时候也没想象中的那么有用，gdb调试本身也不是一个简单的事，而且大部分情况下程序panic会输出到日志中，查日志就能很快查出来，上次为了用这个还是因为项目程序进程离奇失踪，日志也没异常的情况。
