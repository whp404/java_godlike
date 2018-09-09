## Jinfo命令


jinfo可以输出java进程、core文件或远程debug服务器的配置信息。这些配置信息包括JAVA系统参数及命令行参数,如果进程运行在64位虚拟机上，需要指明-J-d64参数，如：jinfo -J-d64 option pid
（恩，就是用来打印一些虚拟机运行参数的？）


### 用法摘要

```
Usage:
    jinfo [option] <pid>
        (to connect to running process)
    jinfo [option] <executable <core>
        (to connect to a core file)
    jinfo [option] [server_id@]<remote server IP or hostname>
        (to connect to remote debug server)

where <option> is one of:
    -flag <name>         to print the value of the named VM flag
    -flag [+|-]<name>    to enable or disable the named VM flag 
    -flag <name>=<value> to set the named VM flag to the given value
    -flags               to print VM flags
    -sysprops            to print Java system properties
    <no option>          to print both of the above
    -h | -help           to print this help message
    
我就挑了几个命令打印一下，其中我的eclipse 的进程号7940    
 C:\Users\Think>jinfo -flags 7940  //命令打印出了我的eclipse虚拟机运行参数
Attaching to process ID 7940, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.91-b14
Non-default VM flags: -XX:CICompilerCount=4 -XX:InitialHeapSize=268435456 -XX:MaxHeapSize=4294967296 -XX:MaxNewSize=1431
306240 -XX:MinHeapDeltaBytes=524288 -XX:NewSize=89128960 -XX:OldSize=179306496 -XX:+UseCompressedClassPointers -XX:+UseC
ompressedOops -XX:+UseFastUnorderedTimeStamps -XX:-UseLargePagesIndividualAllocation -XX:+UseParallelGC
Command line:  -Dosgi.requiredJavaVersion=1.8 -Xms256m -Dosgi.module.lock.timeout=10 -Dorg.eclipse.swt.browser.IEVersion
=10001 -Xmx4096m -clearPersistedState -javaagent:E:\spring-tool-suite-3.9.5.RELEASE-e4.8.0-win32-x86_64\sts-bundle\sts-3
.9.5.RELEASE\lombok.jar  

尝试自己更换了java堆栈的最大值，虽然失败了
C:\Users\Think>jinfo -flag Xmx=5500m 7940
Exception in thread "main" com.sun.tools.attach.AttachOperationFailedException: flag 'Xmx' cannot be changed

        at sun.tools.attach.WindowsVirtualMachine.execute(WindowsVirtualMachine.java:117)
        at sun.tools.attach.HotSpotVirtualMachine.executeCommand(HotSpotVirtualMachine.java:261)
        at sun.tools.attach.HotSpotVirtualMachine.setFlag(HotSpotVirtualMachine.java:234)
        at sun.tools.jinfo.JInfo.flag(JInfo.java:134)
        at sun.tools.jinfo.JInfo.main(JInfo.java:81)
```

## Jstat命令
>jstat(JVM Statistics Monitoring Tool)是用于监控虚拟机各种运行状态信息的命令行工具。他可以显示本地或远程虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据，在没有GUI图形的服务器上，它是运行期定位虚拟机性能问题的首选工具。

命令格式：
 
```
jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]
参数解释：
Option — 选项，我们一般使用 -gcutil 查看gc情况

vmid — VM的进程号，即当前运行的java进程号，当时远程虚拟机的话格式protocol:][//] lvmid [@hostname[:port]/servername]

interval– 间隔时间，单位为秒或者毫秒

count — 打印次数，如果缺省则打印无数次

参数interval和count代表查询间隔和次数，如果省略这两个参数，说明只查询一次。假设需要每250毫秒查询一次进程5828垃圾收集状况，一共查询5次，那命令行如下：
jstat -gc 5828 250 5

```
其中option 参数主要分为三大类：类装载、垃圾收集和运行期编译状况

####  常用命令练习(这里都只打印一次）
**需要提醒的是，我这里使用jdk1.8的所有，永久代貌似变成了元空间（p开头的变成m开头的）**
```

jstat –class<pid> : 显示加载class的数量，及所占空间等信息。

C:\Users\Think>jstat -class 7940
Loaded  Bytes  Unloaded  Bytes     Time
 24012 48449.9        0     0.0     123.16

Loaded 装载的类的数量 Bytes 装载类所占用的字节数 Unloaded 卸载类的数量 Bytes 卸载类的字节数 Time 装载和卸载类所花费的时间
 

---

jstat -compiler <pid>显示VM实时编译的数量等信息。
C:\Users\Think>jstat -compiler 7940
Compiled Failed Invalid   Time   FailedType FailedMethod
   17781      2       0   208.68          1 org/eclipse/ui/internal/menus/LegacyActionPersistence convertA
   
Compiled 编译任务执行数量 Failed 编译任务执行失败数量 Invalid 编译任务执行失效数量 Time 编译任务消耗时间 FailedType 最后一个编译失败任务的类型 FailedMethod 最后一个编译失败任务所在的类及方法


---


jstat -gc <pid>: 可以显示gc的信息，查看gc的次数，及时间。

C:\Users\Think>jstat -gc 7940
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     Y
FGCT     GCT
62976.0 50176.0  0.0   50152.1 819200.0 283429.3  347136.0   119339.8  153088.0 137574.4 21248.0 16937.6
   4      2.431    3.996
```
恩 总结就是c结尾代表容量 U结尾代表已使用（used）
- S0C 年轻代中第一个survivor（幸存区）的容量 (字节) 
- S1C 年轻代中第二个survivor（幸存区）的容量 (字节)
- S0U 年轻代中第一个survivor（幸存区）目前已使用空间 (字节) 
- S1U 年轻代中第二个survivor（幸存区）目前已使用空间 (字节) 
- EC 年轻代中Eden（伊甸园）的容量 (字节) 
- EU 年轻代中Eden（伊甸园）目前已使用空间 (字节) 
- OC Old代的容量 (字节) 
- OU Old代目前已使用空间 (字节) 
- PC Perm(持久代)的容量 (字节) 
- PU Perm(持久代)目前已使用空间 (字节) 
- YGC 从应用程序启动到采样时年轻代中gc次数 
- YGCT 从应用程序启动到采样时年轻代中gc所用时间(s)
- FGC 从应用程序启动到采样时old代(全gc)gc次数 
- FGCT 从应用程序启动到采样时old代(全gc)gc所用时间(s) 
- GCT 从应用程序启动到采样时gc用的总时间(s)


```
jstat -gcutil <pid>:统计gc信息 ，和-gc差不多，就是参数变成了百分比

jstat -gccapacity <pid>:可以显示，VM内存中三代（young,old，metaData）对象的使用和占用大小,一般会显示最小容量，最大容量和当前容量

类似 这一小段 ：GCMN 年轻代(young)中初始化(最小)的大小(字节) NGCMX 年轻代(young)的最大容量 (字节) NGC 年轻代(young)中当前的容量 (字节) S0C 年轻代中第一个survivor（幸存区）的容量 (字节) S1C 年轻代中第二个survivor（幸存区）的容量 (字节) EC 年轻代中Eden（伊甸园）的容量 (字节)


jstat -gcnew <pid>:年轻代对象的信息。
jstat -gcnewcapacity<pid>: 年轻代对象的信息及其占用量。


jstat -gcold <pid>：old代对象的信息。
jstat -gcoldcapacity <pid>: old代对象的信息及其占用量。

jstat -gcpermcapacity<pid>: perm对象的信息及其占用量。


jstat -printcompilation <pid>：当前VM执行的信息。

Compiled 编译任务的数目 Size 方法生成的字节码的大小 Type 编译类型 Method 类名和方法名用来标识编译的方法。类名使用/做为一个命名空间分隔符。方法名是给定类中的方法。上述格式是由-XX:+PrintComplation选项进行设置的

C:\Users\Think>jstat -printcompilation 7940
Compiled  Size  Type Method
   19421     20    1 java/util/Collections$EmptySet forEach
```


## jhat
>jhat(Java Heap Analysis Tool),是一个用来分析java的堆情况的命令。之前的文章讲到过，使用jmap可以生成Java堆的Dump文件。生成dump文件之后就可以用jhat命令，将dump文件转成html的形式，然后通过http访问可以查看堆情况。


### 主要流程如下
- 生成dump 文件：jmap -dump:format=b,file=heapDump 62247
- 解析Java堆转储文件,并启动一个 web server：  jhat heapDump
- 使用jhat命令，就启动了一个http服务，端口是7000
- 然后在访问http://localhost:7000/


具体参考 [Hoils博客](http://www.hollischuang.com/archives/1047)