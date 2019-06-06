# 初识Linux shell

## 什么是Linux

linux可划分为四部分：

- linux内核
- Gnu工具
- 图形化桌面环境
- 应用软件

### linux内核

内核主要负责以下四种功能

- 系统内存管理 （虚拟内存，交换空间）
- 软件程序管理 （init进程，运行级）
- 硬件设备管理 （编译进内核的设备驱动代码，可插入内核的设备驱动模块）（设备文件：字符型设备文件，块设备文件，网络设备文件）
- 文件系统管理 （虚拟文件系统）

### GNU工具

模仿Unix操作系统开发一系列标准的计算机系统工具。

1. 核心GNU工具 （用以处理文件的工具，用以操作文本的工具，用以管理进程的工具）
2. shell （一种特殊的交互式工具。shell的核心是命令行提示符。命令行提示符是shell负责交互的部分。它允许你输入文本命令，然后解释命令，并在内核中执行，也可以将多个shell命令放入文件中作为程序执行。这些文件被称为shell脚本）

| shell |                             描述                             |
| :---: | :----------------------------------------------------------: |
|  ash  |    运行在内存受限环境中的轻量shell，与bash shell完全兼容     |
| korn  |    兼容Bourne shell，支持关联数组和浮点运算等高级编程特性    |
| tcsh  |             将C语言中的一些元素引入到shell脚本中             |
|  zsh  | 结合了bash，tcsh，korn的特性，同时童工高级编程特性，共享历史文件和主题化提示符的高级shell |

### linux桌面环境

1. X Windows系统 （它没有桌面环境供用户操作文件或事开启程序，需要一个建立在X window系统软件之上的桌面环境）
2. KDE桌面
3. GMOME桌面
4. Unity桌面
5. 其他 （Fluxbox, Xfce, JWM, Fvwm. fvwm95）

## Linux发行版

完整的Linux系统包称为发行版。

- 完整的核心Linux发行版
- 特定用途的发行版
- LiveCD测试发行版

### 核心Linux发行版

Slackware，RedHat，Fedora，Gentoo，openSUSE，Debian

### 特定用途的Linux发行版

CentOS，Ubuntu，PCLinuxOS，Mint，dyne:bolic，Puppy Linux

### Linux LiveCD

Knoppix，PCLinuxOS，Ubuntu，Slax，Puppy Linux

# 走进Shell

## 进入命令行

文本命令行界面（command line interface， CLI）

- 控制台终端
- 图形化终端 （终端仿真包）

## 通过Linux控制台终端访问CLI

一般CTRL+ALT+（F1～F7）

tty2：2表示这是虚拟控制台2，tty代表电传打字机（teletypewriter）

```shell
setterm -inversescreen [on off] 
```

设置终端的背景色和文本颜色

```shell
setterm --background [black red green yellow blue magenta cyan white]#设置终端背景色
settern --foreground [black red green yellow blue magenta cyan white]#设置终端前景色
settern -reset #恢复成默认设置并清屏
settern -store #当前前景色和背景色设置成默认
```

## 通过图形化终端仿真访问CLI

Eterm，Final Term，GNOME Terminal...

# 基本的Bash shell命令

## 启动shell

/etc/password文件包含了所有系统用户账户列表，以及每个用户的基本配置信息。

## bash手册

man命令用来访问存储在Linux系统上的手册页面

这些页面是由**分页程序** （pager）来显示的。

|      节       |            描述            |
| :-----------: | :------------------------: |
|     Name      | 显示命令名和一段简短的描述 |
|   Synopsis    |         命令的语法         |
| Configuration |       命令的配置信息       |
|  Description  |      命令的一般性描述      |
|    Options    |        命令选项描述        |
|  Exit Status  |     命令的推出状态提示     |
| Return Value  |        命令的返回值        |
|    Errors     |       命令的错误信息       |
|  Environment  |    描述所使用的环境变量    |
|     Files     |       命令用到的文件       |
|   Versions    |       命令的版本信息       |
| Conforming To |      命令所遵循的标准      |
|     Notes     |      其他有帮助的资料      |
|     Bugs      |     提供提交bug的途径      |
|    Authors    |     命令开发人员的信息     |
|   Copyright   |    命令源代码的版权情况    |
|   See Also    |   与该命令类型的其他命令   |

```shell
man -k [keyword] #关键字搜索手册页
```

手册页还有对应的内容区域（1～9）；

## 浏览文件系统

在Linux PC上安装的第一块硬盘称为**根驱动器**

Linux会在根驱动器上创建一些特别的目录，我们称之为**挂载点**

| 目录   | 用途                                                      |
| ------ | --------------------------------------------------------- |
| /      | 虚拟目录的根目录。通常不会在这里存储文件                  |
| /bin   | 二进制目录，存放许多用户级的GNU工具                       |
| /boot  | 启动目录，存放启动文件                                    |
| /dev   | 设备目录，Linux在这里创建设备节点                         |
| /etc   | 系统配置文件目录                                          |
| /home  | 主目录，Linux在这里创建用户目录                           |
| /lib   | 库目录，存放系统和应用程序的库文件                        |
| /media | 媒体目录，可移动媒体设备的常用挂载点                      |
| /mnt   | 挂载目录，另一个可移动媒体设备的常用挂载点                |
| /opt   | 可选目录，常用于存放第三方软件包和数据文件                |
| /proc  | 进程目录，存放现有硬件及当前进程的相关信息                |
| /root  | root用户的主目录                                          |
| /sbin  | 系统二进制目录，存放本地服务的相关文件                    |
| /run   | 运行目录，存放系统运行时的运行时数据                      |
| /srv   | 服务目录，存放本地服务的相关文件                          |
| /sys   | 系统目录，存放系统硬件信息的相关文件                      |
| /tmp   | 临时目录，可以在该目录中创建和删除临时工作文件            |
| /usr   | 用户二进制目录，大量用户级的GNU工具和数据文件都存储在这里 |
| /var   | 可变目录，用以存放经常变化的文件，比如日志文件。          |

常见的目录名均基于文件系统层级标准（filesystem hierarchy standard，FHS）。

### 遍历目录

```shell
cd /usr/bin
pwd
```

## 文件和目录列表

### 基本列表功能

```shell
ls #当前目录下的文件
ls -F #在目录名后加了 /
ls -a #显示隐藏文件，以.开始的文件名
ls -F -R #递归选项，当前目录下包含的子目录中的文件
ls -FR #与上同
ls -l #产生长列表格式的输出，包含了目录中每个文件的更多相关信息
```

### 长列表

```
drwxr-xr-x 2 christine christine 4096 Apr 22 20:37 Desktop
```

- 文件类型，比如目录（d），文件（-），字符型文件（c）或块设备（b）
- 文件的权限
- 文件的硬连接总数
- 文件的属主的用户名
- 文件的属组的组名
- 文件的大小
- 文件的上次修改时间
- 文件名或目录名

```shell
ls -alF #常用
```

### 过滤输出列表

```shell
ls -l my_script #文件名匹配
ls -l my_scr?pt #？代表一个字符，*代表零个或多个字符
```

在过滤器中使用星号或者问号称为**文件扩展匹配**，通配符正式的名称叫做**元字符通配符**

还有[ai],[a-i],[!a]等通配符

## 处理文件

### 创建文件

```shell
touch filename #会改变文件的修改时间和访问时间
touch -a filename #只会修改文件的访问时间
ls -l filename #默认显示的是修改时间
ls -l --time=atime filename #显示文件的访问时间
```

### 复制文件

```shell
cp source destination #将source文件复制为一个新文件destination
cp -i source destination #强制询问是否需要覆盖已有文件
cp -i source destination/ #复制到现有目录中
ls -Fd #d参数只列出目录本身内容，不列出其中内容
cp -R source/ destination #创建文件夹，并把source中的文件复制到destination文件夹中
cp *script Mod_scripts #可以使用通配符，把当前目录以script结尾的文件复制到Mod_scripts
```

### 制表符自动补全

tab会自动补全匹配的文件和文件路径

### 链接文件

链接是目录中指向文件真实位置的占位符。

- 符号链接（是一个实在的文件，它指向存放在虚拟目录结构中某个地方的另一个文件）
- 硬链接 （创建独立的虚拟文件，包含了雨啊是文件的信息及位置。从根本上而言是同一个文件）

```shell
ln -s datafile sl_datafile #为datafile创建一个符号链接文件sl_datafile
ls -i file #查看文件的inode编号，inode编号是一个由内核分配给文件系统每一个对象的标识数字
ls datafile hl_datafile #为datafile创建一个硬链接文件hl_datafile

8658777686 -rw-r--r--  2 wang  staff  54  6  5 11:25 datafile
8658777686 -rw-r--r--  2 wang  staff  54  6  5 11:25 hl_datafile
8658777712 lrwxr-xr-x  1 wang  staff   8  6  5 11:24 sl_datafile -> datafile
#上面是创建了硬链接和软链接之后文件的输出信息。
```

只能对处于同一存储媒体的文件创建硬链接，要在不同存储媒体的文件之间创建链接只能使用符号链接。

### 重命名文件

重命名文件称为移动（moving）；

mv只影响文件名，不影响时间戳和inode编号

和cp类似，使用**-i**参数，在覆盖已有文件时会得到提示

也可以移动整个文件夹。

### 删除文件

rm尽量加**-i**参数，提示你是不是真的要删除当前文件。**-f**强制删除，谨慎使用。

## 处理目录

### 创建目录

```shell
mkdir -p Dir/Sub_Dir/Under_Dir #同时创建多个目录和子目录用-p参数
```

### 删除目录

rmdir只能删除空目录。因此使用rmdir前需要将目录内的文件删除。

```shell
rm -r #向下进入目录，删除其中的文件，然后再删除目录本身。-r和-R同样的功能
rm -rf #强制删除，谨慎使用
tree dir #tree工具能够以一种美观的方式展示目录。
```

## 查看文件内容

### 查看文件类型

file命令（text文件，目录，链接文件，shell脚本文件，二进制文件）

### 查看整个文件

- cat命令（-n参数会给所有行加上行号，-b只对有文本的行加上行号，-T用^I替换制表符）
- more命令（相比cat实现了分页，通过空格或回车键逐行向前浏览文件）
- less命令（more的升级版，提供一些高级搜索功能）

### 查看部分文件

- tail命令 （默认显示最后十行，-n  num（或者直接-num）来控制显示最后多少行。-f允许其他进程使用该文件时查看文件的内容，并不断显示添加到文件中的内容，可以实现实时监测系统日志）
- head命令（用法和tail相同，只是这个是显示文件的开始行）

# 更多的bash shell命令

## 监测程序

### 探查进程

ps命令

linux系统中使用的GNU ps命令支持三种不同类型的命令行参数

- Unix风格的参数，前面加单破折线
- BSD风格的参数，前面不加破折线
- GNU风格的长参数，前面加双破折线

#### Unix风格的参数

贝尔实验室开发的AT&T Unix系统上原有的ps命令继承下来的。

| 参数        | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| -A          | 显示所有进程                                                 |
| -N          | 显示与指定参数不符的所有进程                                 |
| -a          | 显示除控制进程和无终端进程外的所有进程                       |
| -d          | 显示出控制进程外的所有进程                                   |
| -e          | 显示所有进程                                                 |
| -C cmdlist  | 显示包含在cmdlist列表中的进程                                |
| -G grplist  | 显示组ID再grplist列表中的进程                                |
| -U userlist | 显示属主的用户ID再userlist列表中的进程                       |
| -g grplist  | 显示会话或组ID再grplist列表中的进程<br />（有的发行版中grplist代表会话ID，有的发行版中grplist代表有效组ID） |
| -p pidlist  | 显示PID在pidlist列表中的进程                                 |
| -s sesslist | 显示会话ID再sesslistlieb中的进程                             |
| -t ttylist  | 显示终端ID再ttylist列表中的进程                              |
| -u userlist | 显示有效用户ID再userlist列表中的进程                         |
| -F          | 显示更多额外输出（相对-f参数而言）                           |
| -O format   | 显示默认的输出列以及format列表指定的特定列                   |
| -M          | 显示进程的安全信息                                           |
| -c          | 显示进程的额外调度器信息                                     |
| -f          | 显示完整的格式输出                                           |
| -j          | 显示任务信息                                                 |
| -l          | 显示长列表                                                   |
| -o format   | 仅显示有format指定的列                                       |
| -y          | 不要显示进程标记（process flag，表明进程状态的标记）         |
| -Z          | 显示安全标签信息                                             |
| -H          | 用层级格式来显示进程（树状）                                 |
| -n namelist | 定义了WCHAN列显示的值                                        |
| -w          | 采用宽输出模式，不限宽度显示                                 |
| -L          | 显示进程中的线程                                             |
| -V          | 显示ps命令的版本号                                           |

-f输出包含的信息有：

- UID：启动这些进程的用户
- PID：进程的进程ID
- PPID：父进程的进程号
- C：进程生命周期中的CPU利用率
- STIME：进程启动时的系统时间
- TTY：进程启动时的终端设备
- TIME：运行进程需要的累积CPU时间
- CMD：启动的程序名称

-l长格式输出的信息出了-f的信息还包括：

- F：内核分配给进程的系统标记
- S：进程的状态（O代表正在运行；S代表休眠；R代表可运行，正等待运行；Z代表僵尸化，进程已结束但父进程已不存在；T代表停止）
- PRI：进程的优先级（越大的数字代表越低的优先级）
- NI：谦让度值用来参与决定优先级
- ADDR：进程的内存地址
- SZ：假如进程被换出，所需交换空间的大致大小
- WCHAN：进程休眠的内核函数的地址

#### BSD风格的参数

| 参数       | 描述                                                    |
| ---------- | ------------------------------------------------------- |
| T          | 显示当前终端关联的所有进程                              |
| a          | 显示跟任意终端关联的所有进程                            |
| g          | 显示所有的进程，包括控制进程                            |
| r          | 仅显示运行中的进程                                      |
| x          | 显示所有进程，甚至包括未分配任何终端的进程              |
| U userlist | 显示归userlist列表中某用户ID所有的进程                  |
| p pidlist  | 显示PID在pidlist列表中的所有进程                        |
| t ttylist  | 显示所关联的终端在ttylist列表中的进程                   |
| O format   | 除了默认输出的列之外，还输出由format指定的列            |
| X          | 按过去的Linux i386寄存器格式显示                        |
| Z          | 将安全信息添加的输出中                                  |
| j          | 显示任务信息                                            |
| l          | 采用长模式                                              |
| o format   | 仅显示由format指定的列                                  |
| s          | 采用信号格式显示                                        |
| u          | 采用基于用户的格式显示                                  |
| v          | 采用虚拟内存格式显示                                    |
| N namelist | 定义在WCHAN列中使用的值                                 |
| O order    | 定义显示信息列的顺序                                    |
| S          | 将数值信息从子进程加到夫进程上，比如CPU和内存的使用情况 |
| c          | 显示真实的命令名称（用以启动进程的程序名称）            |
| e          | 显示命令使用的环境变量                                  |
| f          | 用分层格式来显示进程，表明那些进程启动了那些进程        |
| h          | 不显示头信息                                            |
| k sort     | 指定用以将输出排序的列                                  |
| n          | 和WCHEN信息一起显示出来，用数值来表示用户ID和组ID       |
| w          | 为较宽屏幕显示宽输出                                    |
| H          | 将线程按进程来显示                                      |
| m          | 在进程后显示线程                                        |
| L          | 列出所有格式指定符                                      |
| V          | 显示ps命令的版本号                                      |

l命令与UNIX风格不同：

- VSZ：进程在内存中的大小，以千字节（KB）为单位
- RSS：进程在未换出时占用的物理内存
- STAT：代表当前进程状态的双字符状态码

STAT第一个字符采用了和Unix风格S列相同的值，第二个参数进一步说明进程的状态：

- <：该进程运行在高优先级上。
- N：该进程运行在低优先级上
- L：该进程有页面锁定在内存中
- s：该进程时控制进程
- l：该进程时多线程的
- +：该进程运行在前台

#### GNU长参数

| 参数            | 描述                                   |
| --------------- | -------------------------------------- |
| --deselect      | 显示所有的进程，命令行中列出的进程     |
| --Group grplist | 显示组ID在grplist列表中的进程          |
| --User userlist | 显示用户ID在userlist列表中的进程       |
| --group grplist | 显示有效组ID在grplist列表中的进程      |
| --pid pidlist   | 显示PID在pidlist中的进程               |
| --ppid pidlist  | 显示父PID在pidlist列表中的进程         |
| --sid sidlist   | 显示会话ID在sidlist列表中的进程        |
| --tty ttylist   | 显示终端设备好在ttylist列表中的进程    |
| --user userlist | 显示有效用户ID在userlist列表中的进程   |
| --format format | 仅显示由format指定的列                 |
| --context       | 显示额外的安全信息                     |
| --cols n        | 将屏幕宽度设置为n列                    |
| --columns n     | 将屏幕宽度设置为n列                    |
| --cumulative    | 包含已停止的子进程的信息               |
| --forest        | 用层级结构显示出进程和父进程之间的关系 |
| --headers       | 在每页输出中都显示列的头               |
| --no-headers    | 不显示列的头                           |
| --lines n       | 将屏幕高度设置为n排                    |
| --sort order    | 指定将输出按那列排序                   |
| --width n       | 将屏幕宽度设置为n                      |
| --help          | 显示帮助信息                           |
| --info          | 显示调试信息                           |
| --version       | 显示ps命令的版本号                     |

--forest参数可以直观的展示进程的层级信息

### 实时监测

top命令。在top命令运行时键入可改变top的行为。键入f允许你选择对输出进行排序的字段，键入d允许你修改轮询间隔。键入q可以退出top。用户在top命令的输出上有很大的控制权。

### 结束进程

Linux进程信号：

| 信号 | 名称 | 描述                         |
| ---- | ---- | ---------------------------- |
| 1    | HUP  | 挂起                         |
| 2    | INT  | 中断                         |
| 3    | QUIT | 结束运行                     |
| 9    | KILL | 无条件终止                   |
| 11   | SEGV | 段错误                       |
| 15   | TERM | 尽可能终止                   |
| 17   | STOP | 无条件停止运行，但不终止     |
| 18   | TSTP | 停止或暂停，但继续在后台运行 |
| 19   | CONT | 在STOP或TSTP之后恢复执行     |

在linux上有两个命令可以向运行中的进程发出进程信号

- kill命令 （-s参数支持指定其他信号，用信号名或信号值，支持PID）
- killall命令（支持通过进程名来结束进程，支持通配符）

## 监测磁盘空间

### 挂载存储媒体

在使用新的存储媒体之前，需要把它放到虚拟目录下。这项工作称为**挂载**

#### mount命令 

提供如下四部分信息：

- 媒体的设备文件名
- 媒体挂载到虚拟目录的挂载点
- 文件系统类型
- 已挂载媒体的访问状态

```shell
mount -t type device directory #type参数指定了磁盘被格式化的文件系统类型
```

#### umount命令

```shell
umount [directory | device]
```

如果在卸载设备时，系统提示设备繁忙，无法卸载设备，通常是有进程还在访问该设备或使用该设备上的文件。这时可用lsof命令获得使用它的进程信息，然后在应用中停止使用该设备或停止该进程。lsof命令的用法很简单:`lsof /path/to/device/node`，或者`lsof /path/to/mount/point`。

### 使用df命令

可以很方便的查看所有已挂载磁盘的情况

`df -h`输出中的磁盘空间按照用户易读的形式显示，通常用M，G代替兆字节和吉字节。

### 使用du命令

du命令可以显示某个特定目录(默认情况下是当前目录)的磁盘使用情况。

- -c显示所有已列出文件的总大小
- -h按用户易读的格式输出大小
- -s显示每个输出参数的总计

## 处理数据文件

### 排序数据

sort命令会按照会话指定的默认语言的排序规则对文本文件中的数据行排序

默认情况下sort会把数字当作字符来处理。使用**-n**参数告诉sort把数字识别成数字而不是字符。

**-M**按月排序，处理日志的时候可能会用到。

| 单破折线 | 双破折线                              | 描述                                                         |
| -------- | ------------------------------------- | ------------------------------------------------------------ |
| -b       | —ignore-leading-blank                 | 排序时忽略起始的空白                                         |
| -C       | —check=quiet                          | 不排序，如果数据无序也不要报告                               |
| -c       | --check                               | 不排序，检查数据是不是已排序，如不是，报告                   |
| -d       | —dictionary-order                     | 仅考虑空白和字母，不考虑特殊字符                             |
| -f       | —ignore-case                          | 默认情况大写字母排在前面，这个参数忽略大小写                 |
| -g       | --general-number-sort                 | 按通用数值来排序，把值当浮点数来排序，支持科学计数法表示的值 |
| -i       | —ignore-nonprinting                   | 在排序时忽略不可打印字符                                     |
| -k       | —key=POS1[,POS2]                      | 排序从POS1开始，如果指定POS2到POS2结束                       |
| -M       | —month-sort                           | 用三字符月份名按月份排序                                     |
| -m       | --merge                               | 将两个已排序数据文件合并                                     |
| -n       | —numeric-sort                         | 按字符串数值来排序，不转换为浮点数                           |
| -o       | —output=file                          | 将排序结果写入指定文件                                       |
| -R       | —random-sort<br />—random-source=file | 按随机生成的散列表的键值排序<br />指定-R参数用到的随机字节的源文件 |
| -r       | --reverse                             | 反序排序                                                     |
| -S       | --buffer-size=size                    | 指定使用的内存大小                                           |
| -s       | --stable                              | 禁用最后重排序比较                                           |
| -T       | --temporary-directory=DIR             | 指定一个位置来存储临时工作文件                               |
| -t       | —field-seperator=SEP                  | 指定一个用来区分键位置的字符                                 |
| -u       | --unique                              | 和-c参数一起使用时，检查严格排序；不和-c参数一起用时，<br />仅输出第一例相似的两行 |
| -z       | —zero-terminated                      | 用NULL字符作为结尾，而不是换行符                             |

```shell
$ sort -t ':' -k 3 -n /etc/passwd
root:x:0:0:root:/root:/bin/bash #将从0:root:/root:/bin/bash来排序
```

```shell

$ du -sh * | sort -nr #看到目录下的那些文件占用空间最多
1008k   mrtg-2.9.29.tar.gz
972k    bldg1
888k    fbs2.pdf
760k    Printtest
680k    rsync-2.6.6.tar.gz
660k    code
516k    fig1001.tiff
496k    test
496k    php-common-4.0.4pl1-6mdk.i586.rpm
448k    MesaGLUT-6.5.1.tar.gz
400k    plp
```

### 搜索数据

```shell
grep [options] pattern [file]
```

-v 反向搜索（输出不匹配该模式的行）

-n 显示匹配模式的行所在的行号

-c 有多少行含有匹配的模式

-e 指定多个匹配模式

默认情况下，用基本的UNIX风格正则表达式来匹配模式。

egrep是grep的衍生，支持POSIX扩展正则表达式

fgrep是另外一个版本，支持将匹配模式指定为用换行符分隔的一列固定长度的字符串。

### 压缩数据

| 工具     | 文件扩展名 | 描述                                              |
| -------- | ---------- | ------------------------------------------------- |
| bzip2    | .bz2       | 采用burrows-wheeler块排序文本压缩算法和霍夫曼编码 |
| compress | .Z         | 最初的Unix文件压缩工具，很少使用                  |
| gzip     | .gz        | GNU压缩工具，用Lempel-Ziv编码                     |
| zip      | .zip       | Window上PKZIP工具的Unix实现                       |

gzip软件包包含：

- gzip：用来压缩文件
- gzcat：用来查看压缩过的文本文件的内容
- gunzip：用来解压文件

### 归档数据

```shell
tar function [options] object1 object2 ...
```

function参数定义

| 功能 | 长名称               | 描述                                                         |
| ---- | -------------------- | ------------------------------------------------------------ |
| -A   | --concatenate        | 将一个已有tar归档文件追加到另一个已有tar归档文件             |
| -c   | --create             | 创建一个新的tar归档文档                                      |
| -d   | --diff<br />--append | 检查归档文件和文件系统的不同之处<br />从已有tar归档文件中删除 |
| -r   | --append             | 追加文件到已有tar归档文件末尾                                |
| -t   | --list               | 列出已有tar归档文档文件的内容                                |
| -u   | --update             | 将比tar归档文件中已有的同名文件新的文件追加到该tar归档文件中 |
| -x   | --extract            | 从已有tar归档文件中提取文件                                  |

options

| 选项    | 描述                              |
| ------- | --------------------------------- |
| -C dir  | 切换到指定目录                    |
| -f file | 输出结果到文件或设备file          |
| -j      | 将输出重定向给bzip2命令来压缩内容 |
| -p      | 保留所有文件权限                  |
| -v      | 在处理文件时显示文件              |
| -z      | 将输出重定向给gzip命令来压缩内容  |

# 理解shell

## shell的类型

在/etc/passwd文件中，在用户ID记录的第7个字段中列出了默认的shell程序

bash，tcsh，ash，dash，csh，zsh

## shell的父子关系

bash命令行参数

| 参数      | 描述                                        |
| --------- | ------------------------------------------- |
| -c string | 从string中读取命令并处理                    |
| -i        | 启动一个能够接收用户输入的交互shell         |
| -l        | 以登录shell的形式启动                       |
| -r        | 启动一个受限shell，用户会被限制在默认目录中 |
| -s        | 从标准输入中读取命令                        |

使用bash shell命令或是运行shell脚本能够创建出子shell；

进程列表也能够创建出子shell；

### 进程列表

```shell
$ pwd ; ls ; cd /etc ; pwd ; cd ; pwd ; ls #在命令之间加（；）就构成了命令列表
$ (pwd ; ls ; cd /etc ; pwd ; cd ; pwd ; ls)#这些命令包含在括号里就成为了进程列表
```

进程列表是一种命令分组。另一种命令分组是`{ command;}`

使用花括号进行命令分组并不会像进程列表那样创建出子shell；

```shell
echo $BASH_SUBSHELL #如果返回0，表明没有子shell，如果为1或者更大的数字，表明有子shell
```

并非真正的多进程处理；

### 别出心裁的子shell用法

1. 探索后台模式（在命令末尾加上字符&，可将命令置入后台模式，可通过jobs命令查看当前运行在后台模式中的用户进程）
2. 将进程列表置入后台（在最后加&符号，使用tar创建备份文件式可利用后台进程列表完成）
3. 协程（他在后台生成一个子shell，并在子shell中执行命令，要使用coproc）

```shell
coproc sleep 10
coproc My_Job { sleep 10; } #为作业命名
```

这里要注意的是，扩展语法写起来有点麻烦。必须确保在第一个花括号({)和命令名之间有一个空格。还必须保证命令以分号(;)结尾。另外，分号和闭花括号(})之间也得有一个空格。

## 理解shell的内建命令

### 外部命令

它们并不是shell程序的一部分。外部命令程序通常位于/bin、/usr/bin、/sbin或/usr/sbin中

可以使用`which`和`type -a`命令找到它；

当外部命令执行时，会创建出一个子进程。这种操作称为**衍生**（forking）。

就算衍生出子进程或是创建了子shell，你仍然可以通过发送信号与其沟通，这一点无论是在命令行还是在脚本编写中都是极其有用的。发送信号(signaling)使得进程间可以通过信号进行通信。

### 内建命令

内建命令和外部命令的区别在于前者不需要使用子进程来执行。

可以利用type命令来了解某个命令是否是内建的

要注意，有些命令有多种实现。例如echo和pwd既有内建命令也有外部命令。两种实现略有不同。要查看命令的不同实现，使用type命令的-a选项。

命令type -a显示出了每个命令的两种实现。注意，which命令只显示出了外部命令文件。

#### history命令

输入!!，然后按回车键就能够唤出刚刚用过的那条命令来使用。

命令历史记录被保存在隐藏文件.bash_history中，它位于用户的主目录中。这里要注意的是，bash命令的历史记录是先存放在内存中，当shell退出时才被写入到历史文件中。

要实现强制写入，需要使用history命令的-a选项。

如果你打开了多个终端会话，仍然可以使用history -a命令在打开的会话中向.bash_history文件中添加记录。但是对于其他打开的终端会话，历史记录并不会自动更新。这是因为.bash_history文件只有在打开首个终端会话时才会被读取。要想强制重新读取.bash_history文件，更新终端会话的历史记录，可以使用history -n命令。

你可以唤回历史列表中任意一条命令。只需输入惊叹号和命令在历史列表中的编号即可。如（!20）

#### alias命令

```shell
alias -p #查看当前可用的别名
alias li='ls -li' #创建自己的别名，一个别名仅在它所被定义的shell进程中才有效。
```

# 使用Linux环境变量

## 什么是环境变量

bash shell用一个叫作环境变量(environment variable)的特性来存储有关shell会话和工作环境的信息(这也是它们被称作环境变量的原因)。

- 全局变量
- 局部变量

## 全局环境变量

