## jps
>jps位于jdk的bin目录下，其作用是显示当前系统的java进程情况，及其id号。

- jps仅查找**当前用户的Java进程**，而不是当前系统中的所有进程。(类似ps -ef|grep java 而不是 ps -aux|grep java)
### 底层实现原理
jdk中的jps命令可以显示当前运行的java进程以及相关参数，它的实现机制如下：
>java.io.tmpdir 可以-Djava.io.tmpdir=xxx给jvm设置,如果想显示当前tmpdir的位置，可以System.out.println(System.getProperty("java.io.tmpdir"));


java程序在启动以后，会在**java.io.tmpdir**指定的目录下，就是临时文件夹里，生成一个类似于hsperfdata_User的文件夹，这个文件夹里（在Linux中为/tmp/hsperfdata_{userName}/），有几个文件，名字就是java进程的pid，因此列出当前运行的java进程，只是把这个目录里的文件名列一下而已。 至于系统的参数什么，就可以解析这几个文件获得。

### jps参数
- -m 输出传递给main 方法的参数，在嵌入式jvm上可能是null
- -l 输出应用程序main class的完整package名 或者 应用程序的jar文件完整路径名
- -v 输出传递给JVM的参数 在这里，在启动main方法的时候，类似-Dfile.encoding=UTF-8：


### jps失效的原因
>现象：用ps -ef|grep java能看到启动的java进程，但是用jps查看却不存在该进程的id。待会儿解释过之后就能知道在该情况下，jconsole、jvisualvm可能无法监控该进程，其他java自带工具也可能无法使用
- 磁盘读写、目录权限问题 若该用户没有权限写/tmp目录或是磁盘已满，则无法创建/tmp/hsperfdata_userName/pid文件。或该文件已经生成，但用户没有读权限
- 临时文件丢失，被删除或是定期清理 对于linux机器，一般都会存在定时任务对临时文件夹进行清理，导致/tmp目录被清空。常用的可能定时删除临时目录的工具**为crontab、redhat的tmpwatch、ubuntu的tmpreaper等等**
- java进程信息文件存储地址被设置，不在/tmp目录下 上面我们在介绍时说默认会在/tmp/hsperfdata_userName目录保存进程信息，但由于以上1、2所述原因，可能导致该文件无法生成或是丢失**，所以java启动时提供了参数(-Djava.io.tmpdir)，可以对这个文件的位置进行设置，而jps、jconsole都只会从/tmp目录读取**，而无法从设置后的目录读物信息

[参考博文 ：H神博客](hhttp://www.hollischuang.com/archives/105)


## javap指令
javap命令反汇编一个或多个类文件。 其输出取决于使用的选项。 如果没有使用任何选项，javap将打印传递给它的类的包，受保护和公共字段以及方法。 javap将其输出打印到stdout。


### javap参数
- -c为类中的每个方法打印出反汇编代码，即包含Java字节码的指令。
- -classpath path 指定javap寻找class文件的地方
- -bootclasspath path 指定系统加载器加载路径
-extdirs dirs 指定拓展加载器加载文件路径
