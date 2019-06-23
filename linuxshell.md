# 理解Linux文件权限

## linux的安全性

UID来跟踪用户的权限。

### /etc/password文件

root的UID为0.

为各种的功能创建不同的用户账户，这些账户不是真正的用户，称作系统账户。

所有运行在后台的服务都需要用一个系统用户账户登录到Linux系统上。

Linux为系统账户预留了500以下的UID值。普通账户一般会从500开始。

```
mysql:x:27:27:MySQL Server:/var/lib/mysql:/bin/bash 
```

- 登录用户名
- 用户密码
- 用户账户的UID
- 用户账户的组ID
- 用户账户的文本描述
- 用户HOME目录的位置
- 用户的默认shell

现在大多数Linux系统把用户密码保存在另一个单独的文件中。/etc/shadow

只有特定的程序才能访问这个文件。

### /etc/shadow文件

只有root用户才能访问

```
rich:$1$.FfcK0ns$f1UgiyHQ25wrB/hykCn020:11627:0:99999:7::: 
```

- 与/etc/passwd中的登录名字段对应的登录名
- 加密后的密码
- 自从上次修改密码后过去的天数密码
- 多少天后才能更改密码
- 多少天后必须更改密码
- 密码过期前多少天提醒用户更改密码
- 密码过期后多少天禁用用户账户
- 用户账户被禁用的日期
- 预留字段给将来使用

### 添加新用户

useradd

```
# /usr/sbin/useradd -D #/usr/sbin可能不在path环境变量中，-D查看默认值
GROUP=100
HOME=/home
INACTIVE=-1
EXPIRE=
SHELL=/bin/bash
SKEL=/etc/skel
CREATE_MAIL_SPOOL=yes 
```

- 新用户会被添加到GID为100的公共组
- 新用户的HOME目录位于/home/loginname
- 系用户账户密码在过期后不会被禁用
- 新用户账户未被设置过期日期
- 新用户账户将bash shell作为默认shell
- 系统会将/etc/skel目录下的内容复制到用户的HOME目录下
- 系统为该用户账户在mail下闯进啊一个用于接收邮件的文件

```
# useradd -m test #默认不创建home目录，-m会创建
```

可以在-D选项后跟上一个指定的值来修改系统默认的新用户设置。

### 删除用户

userdel

默认删除/etc/passwd文件中的用户信息，不会删除系统中属于该账户的任何文件。

-r 会删除用户的HOME目录以及邮件目录

### 修改用户

| 命令 | 描述 |
| ---- | ---- |
|usermod| 修改用户账户的字段，还可以指定主要组以及附加组的所属关系|
|passwd |修改已有用户的密码|
|chpasswd |从文件中读取登录名密码对，并更新密码|
|chage |修改密码的过期日期|
|chfn |修改用户账户的备注信息|
|chsh |修改用户账户的默认登录shell |

#### usermod

参数大部分跟useradd命令参数一样。

- -l修改用户的登录名
- -L锁定用户，使用户无法登录
- -p修改用户账户的密码
- -U解除锁定，使用户能够登录

#### passwd和chpasswd

```
#passwd test #默认该当前用户
```

chpasswd可以从标准输入自动读取，用户名和密码对列表，给密码加密，然后为用户账户设置。

userid\:password

```
# chpasswd < users.txt
```

####  chsh、chfn和chage

```
# chsh -s /bin/csh test 
```

chfn命令会将用于
Unix的finger命令的信息存进备注字段，而不是简单地存入一些随机文本（比如名字或昵称之类的），或是将备注字段留空。finger命令可以非常方便地查看Linux系统上的用户信息。

```
# finger rich
Login: rich Name: Rich Blum
Directory: /home/rich Shell: /bin/bash
On since Thu Sep 20 18:03 (EDT) on pts/0 from 192.168.1.2
No mail.
No Plan.
# 
```

chage命令用来帮助管理用户账户的有效期

这些命令可以看作是提供遍历的方式，修改etc/passwd中的每个对应项的信息。

### 使用Linux组

方便共享资源

有些Linux发行版会创建一个组，把所有用户都当作这个组的成员。
有些发行版会为每个用户创建单独的一个组，

#### /etc/group文件

```
root:x:0:root
bin:x:1:root,bin,daemon
daemon:x:2:root,bin,daemon
sys:x:3:root,bin,adm
adm:x:4:root,adm,daemon
rich:x:500:
mama:x:501:
katie:x:502:
jessica:x:503:
mysql:x:27:
test:x:504: 
```

- 组名
- 组密码
- GID
- 属于该组的用户列表

当一个用户在/etc/passwd文件中指定某个组作为默认组时，用户账户不会作为该组成员再出现在/etc/group文件中。

#### 创建新组

groupadd

```
# /usr/sbin/groupadd shared 
```

```
# /usr/sbin/usermod -G shared rich #添加用户到组
```

如果加了-g选项，指定的组名会替换掉该账户的默认组。-G选项则将该组添加到用户的属组的列表里，不会影响默认组。

如果更改了已登录系统账户所属的用户组，该用户必须登出系统后再登录，组关系的更
改才能生效。

#### 修改组

groupmod命令可以修改已有组的GID（加-g选项）或组名（加-n选项）。

### 理解文件权限

#### 使用文件权限符

- -代表文件
- d代表目录
- l代表链接
- c代表字符型设备
- b代表块设备
- n代表网络设备

之后三组三字符的编码，每一组定义了三种访问权限

- r代表对象是可读的
- w代表对象是可写的
- x代表对象是可执行的

三组权限分别对应对象的三个安全级别

- 对象的属主
- 对象的属组
- 系统的其他的用户

#### 默认文件权限

umask命令设置所创建文件和目录的默认权限

umask值只是掩码

```
umask 026 #文件权限为666-026=640，文件夹权限为777-026=751
```

### 改变安全性设置

#### 改变权限

```
chmod options mode file
```

mode参数可以使用八进制模式和符号模式进行安全性设置。

符号模式的格式

```
[ugoa...] [+-=][rwxXstugo...]
```

第一组

- u代表用户
- g代表组
- o代表其他
- a代表上述所有

第二组

- +表示在现有基础上增加权限
- -标识在现有权限基础上移除权限
- =将权限设置成后面的值

第三组

- rwx标识读写执行
- X如果对象是目录或者它已有执行权限，赋予执行权限
- s执行时重新设置UID或者GID
- t保留文件或目录
- u将权限设置为跟属主一样
- g将权限设置为跟数组一样
- o将权限设置为跟其他用户一样

#### 改变所属关系

chown命令用来改变文件的属主，
chgrp命令用来改变文件的默认属组。

```
chown options owner[.group] file
```

只有root用户能够改变文件的属主。任何属主都可以改变文件的属组，但前提是属主必须是原属组和目标属组的成员。

chgrp命令可以更改文件或目录的默认属组。

### 共享文件

Linux系统上共享文件的方法是创建组

三个额外的信息位

- 设置用户ID（SUID）当文件被用户使用时，程序会以文件属主的权限运行。
- 设置组ID（SGID）对文件来说，程序会以文件属组的权限运行；对目录来说，目录中创建的新文件会以目录的默认属组作为默认属组。
- 粘着位：进程结束后文件还驻留在内存中。

SGID可通过chmod命令设置。它会加到标准3位八进制值之前（组成4位八进制值），或者在符号模式下用符号s。

# 管理文件系统

## 探索Linux文件系统

### 基本的Linux文件系统

#### ext系统

扩展文件系统（extended filesystem）

采用索引节点来存放虚拟目录中所存储文件的信息。

索引节点系统在每个物理设备中创建一个单独的表（称为索引节点表）来存储这些文件的信息。

### ext2文件系统

ext2文件系统扩展了索引节点表的格式来保存系统上每个文件的更多信息。

添加了创建时间值、修改时间值和最后访问时间值来帮助系统管理员追踪文件的访问情况。ext2文件系统还将允许的最大文件大小增加到了2 TB（在ext2的后期版本中增加到了32 TB），

### 日志文件系统

三种日志方法

- 数据模式 风险低，性能差
- 有序模式 风险和性能良好的折中
- 回写模式 风险高，但比不用日志好

#### ext3文件系统

默认使用有序模式的日志功能

无法恢复误删的文件，没有内建的数据压缩功能，不支持加密文件

#### ext4文件系统

支持压缩和加密，还有**区段**特性。引入了**块预分配技术**

#### Reiser文件系统

只支持回写模式，可以调整文件系统的大小，还有**尾部压缩技术**

#### JFS文件系统

采用有序日志方法，采用基于区段的文件分配

#### XFS文件系统

回写模式，可以调整文件系统大小，只能扩大不能缩小。

### 写时复制文件系统

（cory-in-write，COW）如果要修改数据，会使用克隆或可写快照。修改过的数据
并不会直接覆盖当前数据，而是被放入文件系统中的另一个位置上。即便是数据修改已经完成，之前的旧数据也不会被重写。

#### ZFS文件系统

没有GFL许可

#### Btrf文件系统

B树文件系统，具有稳定性、易用性以及能够动态调整已挂载文件系统的大小

## 操作文件系统

### 创建分区

fdisk

主分区可以被文件系统直接格式化，而扩展分区则只能容纳其他逻辑分区。扩展分区出现的原因是每个存储设备上只能有4个分区。可以通过创建多个扩展分区，然后在扩展分区内创建创建逻辑分区进行扩展。

### 创建文件系统

要想知道某个文件系统工具是否可用，可以使用type命令

```
$ type mkfs.ext4
mkfs.ext4 is /sbin/mkfs.ext4
$
$ type mkfs.btrfs
-bash: type: mkfs.btrfs: not found
$ 
```

创建文件系统

挂载到虚拟目录下的某个挂载点

这种挂载文件系统的方法只能临时挂载文件系统。当重启Linux系统时，文件系统并不会自动挂载。要强制Linux在启动时自动挂载新的文件系统，可以将其添加到/etc/fstab文件。

### 文件系统的检查与修复

```
fsck options filesystem
```

## 逻辑卷管理

### 逻辑卷管理布局

物理卷

卷组

### Linux中的LVM

- 快照 LVM1只提供可读快照，LVM2提供可读写快照
- 条带化 LVM2提供，LVM条带化不提供用来创建容错环境的校验信息
- 镜像

### 使用LVM

#### 定义物理卷

fdisk 分区类型8e

pvcreate定义了用于物理卷的物理分区。它只是简单地将分区标记成Linux LVM系统中的分区而已

如果你想查看创建进度的话，可以使用pvdisplay命令来显示已创建的物理卷列表。

#### 创建卷组

```
$ sudo vgcreate Vol1 /dev/sdb1
 Volume group "Vol1" successfully created
$ 
```

查看新创建的卷组的细节，可用vgdisplay命令

#### 创建逻辑卷

```
$ sudo lvcreate -l 100%FREE -n lvtest Vol1
 Logical volume "lvtest" created
$ 
```

查看你创建的逻辑卷的详细情况，可用lvdisplay命令。

#### 创建文件系统

```
$ sudo mkfs.ext4 /dev/Vol1/lvtest 
```

之后就可以挂载到虚拟目录中。

##### 修改LVM

| 命令 | 功能 |
| ---- | ---- |
|vgchange| 激活和禁用卷组|
|vgremove |删除卷组|
|vgextend |将物理卷加到卷组中|
|vgreduce |从卷组中删除物理卷|
|lvextend |增加逻辑卷的大小|
|lvreduce |减小逻辑卷的大小|

# 安装软件程序

包管理系统（package management system，PMS）

## 包管理基础

PMS用数据库记录

- Linux系统上已经安装了什么软件包
- 每个包安装了什么文件
- 每个已安装软件包的版本

PMS两种住哟啊的PMS工具是dpkg和rpm

## 基于Debian的系统

### 使用aptitude管理软件包

```shell
aptitude show package_name #显示某个特定包的详细信息
dpkg -L package_name #显示所有跟某个特定软件包相关的所有文件的列表
dpkg --search absolute_file_name #查找某个特定文件属于哪个软件包，在使用的时候必须用绝对文件路径。
```

### 用 aptitude 安装软件包

```
aptitude search package_name #搜索特定的软件包
aptitude install package_name #安装软件包
```

在每个包名字之前都有一个p或i。如果看到一个i，说明这个包现在已经安装到了你
的系统上了。如果看到一个p或v，说明这个包可用，但还没安装。

### 用 aptitude 更新软件

```
aptitude safe-upgrade 
aptitude full-upgrade
aptitude dist-upgrade 
```

### 用 aptitude 卸载软件

要想只删除软件包而不删除数据和配置文件，可以使用aptitude的remove选项。要删除软件包和相关的数据和配置文件，可用purge选项。

### aptitude 仓库

aptitude默认的软件仓库位置是在安装Linux发行版时设置的。具体位置存储在文件 /etc/apt/sources.list中。

## 基于RedHat系统

### 列出已安装包

```
yum list installed
yum provides filename #要找出系统上的某个特定文件属于哪个软件包
```

### 用yum安装软件

```
yum install packge_name
yum locolinstall packge_name.rpm
```

### 用yum更新软件

```
yum list updates
yum update package_name
```

### 用yum卸载软件

```
yum remove package_name #保留配置文件和数据文件
yum erase package_name
```

### 处理损坏的包依赖关系

```
yum clean all
yum update

yum deplist package_name

yum update --skip-broken 
```

### yum 软件仓库

yum的仓库定义文件位于/etc/yum.repos.d

## 从源码安装

```
# tar -zxvf sysstat-11.1.1.tar.gz
$ cd sysstat-11.1.1
$ ls #看到README或AAAREADME文件。读这个文件非常重要。该文件中包含了软件安装所需要的操作
# ./configure
# make
# make install #安装到Linux系统中常用的位置上
```

# 使用编辑器

## Vim编辑器

```
 readlink -f /usr/bin/vi #readlink –f命令。它能够立刻找出链接文件的最后一环。
```

## Nano编辑器

## Emacs编辑器

## KDE编辑器

- KWrite编辑器
- Kate编辑器

## GNOME编辑器

- gedit编辑器

# 构建基本脚本

## 使用多个命令

如果要两个命令一起运行，可以把它们放在同一行中，彼此间用分号隔开。

## 创建 shell 脚本文件

在创建shell脚本文件时，必须在文件的第一行指定要使用的shell。其格式为：

```
#!/bin/bash 
```

运行脚本时

- 将shell脚本文件所处的目录添加到PATH环境变量中；
-  在提示符中用绝对或相对文件路径来引用shell脚本文件。

## 显示消息

echo

想把文本字符串和命令输出显示在同一行中，可以用echo语句的-n参数。

## 使用变量

### 环境变量

```
$HOME
$(variable)
```

### 用户变量

定义变量允许临时存储数据并在整个脚本中使用

引用一个变量值时需要使用美元符，而引用变量来对其进行赋值时则不要使用美元符

### 命令替换

- 反引号（`）
- $()格式

命令替换会创建一个子shell来运行对应的命令。子shell（subshell）是由运行该脚本的shell所创建出来的一个独立的子shell（child shell）。正因如此，由该子shell所执行命令是无法使用脚本中所创建的变量的。
在命令行提示符下使用路径./运行命令的话，也会创建出子shell；要是运行命令的时候
不加入路径，就不会创建子shell。如果你使用的是内建的shell命令，并不会涉及shell。在命令行提示符下运行脚本时一定要留心！

## 重定向输入和输出

### 输出重定向

输出发送到一个文件

```
command > outputfile
command >> outputfile #追加数据
```

### 输入重定向

```
command < outputfile
```

内联输入重定向

```
command << marker
data
marker
```

## 管道

将一个命令的输出直接重定向到另一个命令。这个过程叫作管道连接（piping）

```
command1 | command2
```

## 执行数学运算

### expr命令

```
$ expr 1 + 5
6 
```

对于那些容易被shell错误解释的字符，在它们传入expr命令之前，需要使用shell的转义字符（反斜线）将其标出来。

### 使用方括号

bash shell数学运算符只支持整数运算。

### 浮点解决方案

- bc内建工具
- 利用重定向在脚本中使用bc（使用内联输入冲顶向更加的清晰）

```
$ cat test12
#!/bin/bash
var1=10.46
var2=43.67
var3=33.2
var4=71
var5=$(bc << EOF
scale = 4
a1 = ( $var1 * $var2)
b1 = ($var3 * $var4)
a1 + b1
EOF
)
echo The final answer for this mess is $var5
$ 
```

## 退出脚本

shell中运行的每个命令都使用退出状态码（exit status）告诉shell它已经运行完毕。退出状态码是一个0～255的整数值，在命令结束运行时由命令传给shell。可以捕获这个值并在脚本中使用。

### 查看退出状态码

Linux提供了一个专门的变量$?来保存上个已执行命令的退出状态码。

一个成功结束的命令的退出状态码是0。如果一个命令结束时有错误，退出状态
11.8 退出脚本 码就是一个正数值。

### exit命令

exit命令跟上常数或者变量返回自己定义的退出状态码。

# 使用结构化命令

## 使用if-then语句

```
if command
then
	commands
fi
```

bash shell的if语句会运行if后面的那个命令。如果该命令的退出状态码是0（该命令成功运行），位于then部分的命令就会被执行。如果该命令的退出状态码是其他值，then部分的命令不会运行。

```
if command; then
commands
fi #另一种写法
```

## if-then-else语句

```
if command
then
 commands
else
 commands
fi
```

## 嵌套if

```
if command1
then
 commands
elif command2
then
 more commands
fi 
```

在elif语句中，紧跟其后的else语句属于elif代码块。它们并不属于之前的if-then代码块。

## test命令

如果test命令中列出的条件成立，test命令就会退出并返回退出状态码0。

```
if test condition
then
 commands
fi 
```

bash shell提供了另一种条件测试方法，无需在if-then语句中声明test命令。

```
if [ condition ]
then
commands
fi
```

方括号定义了测试条件。注意，第一个方括号之后和第二个方括号之前必须加上一个空格，否则就会报错。

test命令可以判断三类条件

- 数值比较
- 字符串比较
- 文件比较

### 数值比较

|比较|描述|
|-------|------|
|n1 -eq n2 |检查n1是否与n2相等|
|n1 -ge n2 |检查n1是否大于或等于n2|
|n1 -gt n2 |检查n1是否大于n2|
|n1 -le n2 |检查n1是否小于或等于n2|
|n1 -lt n2 |检查n1是否小于n2|
|n1 -ne n2| 检查n1是否不等于n2|

bash shell只能处理整数。如果你只是要通过echo语句来显示这个结果，那没问题。但是，在基于数字的函数中就不行了，例如我们的数值测试条件。最后一行就说明我们不能在test命令中使用浮点值。

### 字符串比较

|比较| 描述|
|---|---|
|str1 = str2 |检查str1是否和str2相同|
|str1 != str2| 检查str1是否和str2不同|
|str1 < str2| 检查str1是否比str2小|
|str1 > str2 |检查str1是否比str2大|
|-n str1| 检查str1的长度是否非0|
|-z str1| 检查str1的长度是否为0 |

#### 字符串相等性

比较测试会将所有的标点和大小写情况都考虑在内。

#### 字符串顺序

- 大于号和小于号必须转义
- 大于和小于顺序和sort命令采用的不同

在比较测试中，大写字母被认为是小于小写字母的。但sort命令恰好相反。当你将同样的字符串放进文件中并用sort命令排序时，小写字母会先出现。这是由各个命令使用的排序技术不同造成的。

#### 字符串大小

空的和未初始化的变量会对shell脚本测试造成灾难性的影响。如果不是很确定一个变量的内容，最好在将其用于数值或字符串比较之前先通过-n或-z来测试一下变量是否含有值。

### 文件比较

|比 较| 描 述|
|---|---|
|-d file| 检查file是否存在并是一个目录|
|-e file| 检查file是否存在（文件和目录） |
|-f file |检查file是否存在并是一个文件|
|-r file |检查file是否存在并可读|
|-s file |检查file是否存在并非空|
|-w file |检查file是否存在并可写|
|-x file |检查file是否存在并可执行|
|-O file |检查file是否存在并属当前用户所有|
|-G file |检查file是否存在并且默认组与当前用户相同|
|file1 -nt file2 |检查file1是否比file2新（不会检查文件是否存在）|
|file1 -ot file2 |检查file1是否比file2旧（不会检查文件是否存在）|

## 复合条件测试

有两种布尔运算符可用：

-  [ condition1 ] && [ condition2 ]
-  [ condition1 ] || [ condition2 ]

## if-then的高级 特性

- 用于数学表达式的双括号
- 用于高级字符串处理功能的双方括号

### 使用双括号

```
(( expression )) 
```

|符 号| 描 述|
|---|---|
|val++| 后增|
|val-- |后减|
|++val| 先增|
|--val |先减|
|! |逻辑求反|
|~ |位求反|
|** |幂运算|
|<< |左位移|
|\>\>|右位移|
|& |位布尔和|
|\| |位布尔或|
|&&| 逻辑和|
|\|\|| 逻辑或|

注意，不需要将双括号中表达式里的大于号转义。这是双括号命令提供的另一个高级特性。

### 使用双方括号

```
[[ expression ]]
```

双方括号在bash shell中工作良好。不过要小心，不是所有的shell都支持双方括号。

提供了模式匹配的特性。

使用双等号（==）。双等号将右边的字符串（r*）视为一个模式，
并应用模式匹配规则。

## case命令

```
case variable in
pattern1 | pattern2) commands1;;
pattern3) commands2;;
*) default commands;;
esac
```

# 更多的结构化命令

## for命令

```shell
for var in list
do
 commands
done 
```

### 读取列表中的值

`$var`变量的值会在shell脚本的剩余部分一直保持有效。它会一直保持最后一次迭代的值（除非你修改了它）

### 读取列表中的复杂值

对于引号的需要注意使用转义字符。

for默认是通过空格分隔的，当使用带有空格的词时候，需要用双引号把这些值圈起来。

### 从变量读取列表

将一系列值都集中存储在了一个变量中，然后需要遍历变量中的整个列表。也可以通过for命令完成这个任务。

### 从命令读取值

可以用命令替换来执行任何能产生输出的命令，然后在for命令中使用该命令的输出。

### 更改字段分隔符

环境变量`IFS` 叫做**内部字段分隔符**

默认情况下，bash shell会将下列字段当作字段分隔符：

- 空格
- 制表符
- 换行符

如果bash shell在数据中看到了这些字符中的任意一个，它就会假定这表明了列表中一个新数据字段的开始。

```
IFS=$'\n' #修改IFS的值
IFS=: #冒号为分隔符
IFS=$'\n':;" #换行符，冒号，逗号，双引号作为字段分隔符。
```

### 用通配符读取目录

用for命令来自动遍历目录中的文件。必须在文件名或路径名中使用通配符。它会强制shell使用文件扩展匹配。

可以在for命令中列出多个目录通配符，将目录查找和列表合并进同一个for语句。

## C语言风格的for命令

### C语言的for命令

```
for (( variable assignment ; condition ; iteration process ))
for (( a = 1; a < 10; a++ ))
```

有些部分没有遵循bash shell标准的for命令

- 变量复制可以有空格
- 条件中的变量不以美元符开头
- 迭代过程的算式未用expr命令格式

### 使用多个变量

可以为每个变量定义不同的迭代过程。尽管可以使用多个变量，但你只能在for循环中定义一种条件。

```
#!/bin/bash
# multiple variables
for (( a=1, b=10; a <= 10; a++, b-- ))
do
 echo "$a - $b"
done 
```

## while命令

只要定义的测试命令返回的是退出状态码0。它会在每次迭代的一开始测试test命令。

### while的基本格式

```
while test command
do
 other commands
done 
```

### 使用多个测试命令

注意，每个测试命令都出现在单独的一行上。

```
var1=10
while echo $var1
 [ $var1 -ge 0 ]
do
 echo "This is inside the loop"
 var1=$[ $var1 - 1 ]
done
```

## until命令

until命令要求你指定一个通常返回非零退出状态码的测试命令。

```
until test commands
do 
 other commands
done 
```

## 嵌套循环

whle，for，until可以循环嵌套

## 循环处理文件数据

- 使用嵌套循环
- 修改IFS环境变量

```shell
#!/bin/bash
# changing the IFS value
IFS.OLD=$IFS
IFS=$'\n'
for entry in $(cat /etc/passwd)
do
 echo "Values in $entry –"
 IFS=:
 for value in $entry
 do
 echo " $value"
 done
done
$ 
```

## 循环控制

- break命令
- continue命令

### break命令

- 跳出单个循环
- 跳出内部循环
- 跳出外部循环 break n；其中n指定了要跳出的循环层级。

### continue命令

continue命令可以提前中止某次循环中的命令，但并不会完全终止整个循环。支持continue n。

## 处理循环的输出

在shell脚本中，你可以对循环的输出使用管道或进行重定向。这可以通过在done命令
之后添加一个处理命令来实现。

这种方法同样适用于将循环的结果管接给另一个命令。利用管道（pipe）

## 实例

### 查找可执行文件列表

```shell
$ cat test25
#!/bin/bash
# finding files in the PATH
IFS=:
for folder in $PATH
do
 echo "$folder:"
 for file in $folder/*
 do
 if [ -x $file ]
 then
 echo " $file"
 fi
 done
done
$ 
```

### 创建多个用户账户

```
$ cat test26 
#!/bin/bash
# process new user accounts
input="users.csv"
while IFS=',' read -r userid name
do
 echo "adding $userid"
 useradd -c "$name" -m $userid
done < "$input"
$ 
#read命令会自动读取.csv文本文件的下一行内容，所以不需要专门再写一个循环来处理。
```

# 处理用户输入

## 命令行参数

向shell脚本传递数据的最基本方法是使用命令行参数。

### 读取参数

bash shell会将一些称为位置参数（positional parameter）的特殊变量分配给输入到命令行中的所有参数。这也包括shell所执行的脚本名称。位置参数变量是标准的数字：$0是程序名，$1是第一个参数，$2是第二个参数，依次类推，直到第九个参数$9。

每个参数都是用空格分隔的，所以shell会将空格当成两个值的分隔符。要在参数值中包含空格，必须要用引号（单引号或双引号均可）。

如果脚本需要的命令行参数不止9个，你仍然可以处理，但是需要稍微修改一下变量名。在第9个变量之后，你必须在变量数字周围加上花括号，比如${10}。

### 读取脚本名

```
$ cat test5.sh
#!/bin/bash
# Testing the $0 parameter
#
echo The zero parameter is set to: $0
#
$
$ bash test5.sh
The zero parameter is set to: test5.sh
$ ./test5.sh
The zero parameter is set to: ./test5.sh
$ bash /home/Christine/test5.sh
The zero parameter is set to: /home/Christine/test5.sh
```

basename命令会返回不包含路径的脚本名。

```
$ cat test5b.sh
#!/bin/bash
# Using basename with the $0 parameter
#
name=$(basename $0)
echo
echo The script name is: $name
#
$ bash /home/Christine/test5b.sh
The script name is: test5b.sh
$
$ ./test5b.sh
The script name is: test5b.sh
$ 
```

### 测试参数

在使用参数前一定要检查其中是否存在数据。（-n）

## 特殊参数变量

### 参数统计

特殊变量`$#`含有脚本运行时携带的命令行参数的个数。可以在脚本中任何地方使用这个特殊变量，就跟普通变量一样。

获取最后一个参数，不用直到参数的个数用`${!#}`

### 抓取所有数据

`$*`和`$@`变量可以用来轻松访问所有的参数。这两个变量都能够在单个变量中存储所有的命令行参数。
`$*`变量会将命令行上提供的所有参数当作一个单词保存。这个单词包含了命令行中出现的每一个参数值。基本上`$*`变量会将这些参数视为一个整体，而不是多个个体。
`$@`变量会将命令行上提供的所有参数当作同一字符串中的多个独立的单词。这样你就能够遍历所有的参数值，得到每个参数。这通常通过for命令完成。

```
$ cat test12.sh
#!/bin/bash 
# testing $* and $@
#
echo
count=1
#
for param in "$*"
do
 echo "\$* Parameter #$count = $param"
 count=$[ $count + 1 ]
done
#
echo
count=1
#
for param in "$@"
do
 echo "\$@ Parameter #$count = $param"
 count=$[ $count + 1 ]
done
$
$ ./test12.sh rich barbara katie jessica
$* Parameter #1 = rich barbara katie jessica
$@ Parameter #1 = rich
$@ Parameter #2 = barbara
$@ Parameter #3 = katie
$@ Parameter #4 = jessica
$
```

## 移动变量

用shift命令时，默认情况下它会将每个参数变量向左移动一个位置。所以，变量$3的值会移到$2中，变量$2的值会移到$1中，而变量$1的值则会被删除（注意，变量$0的值，也就是程序名，不会改变）。

要给shift命令提供一个参数，可以指明要移动的位置数

使用shift命令的时候要小心。如果某个参数被移出，它的值就被丢弃了，无法再恢复。

## 处理选项

### 查找选项

在命令行上，它们紧跟在脚本名之后，就跟命令行参数一样。可以像处理命令行参数一样处理命令行选项。

#### 处理简单选项

case语句检查每个参数是不是有效选项。如果是的话，就运行对应case语句中的命令。

#### 分离参数项

shell会用双破折线来表明选项列表结束。

当脚本遇到双破折线时，它会停止处理选项，并将剩下的参数都当作命令行参数。

#### 处理带值的选项

```
$ ./testing.sh -a test1 -b -c -d test2
```

如果你想将多个选项放进一个参数中时，它就不能工作了。

### 使用getopt命令

#### 命令的格式

```
getopt optstring parameters
```

首先，在optstring中列出你要在脚本中用到的每个命令行选项字母。然后，在每个需要参数值的选项字母后加一个冒号。

如果指定了一个不在optstring中的选项，默认情况下，getopt命令会产生一条错误消息。如果想忽略这条错误消息，可以在命令后加-q选项。

#### 在脚本中使用getopt

```
set -- $(getopt -q ab:cd "$@")
```

set命令的选项之一是双破折线（--），它会将命令行参数替换成set命令的命令行值；

getopt命令并不擅长处理带空格和引号的参数值。它会将空格当作参数分隔符，而不是根据双引号将二者当作一个参数。

### 使用更高级的getopts

```
getopts optstring variable
```

每次调用它时，它一次只处理命令行上检测到的一个参数。处理完所有的参数后，它会退出并返回一个大于0的退出状态码。这让它非常适合用解析命令行所有参数的循环中。

要去掉错误消息的话，可以在optstring之前加一个冒号。

如果选项需要跟一个参数值，OPTARG环境变量就会保存这个值

getopts命令解析命令行选项时会移除开头的单破折线，所以在case定义中不用单破折线。

- 可以在参数值中包含空格
- 将选项字母和参数值放在一起使用，而不用加空格。
- 将命令行上找到的所有未定义的选项统一输出成问号。

getopts命令知道何时停止处理选项，并将参数留给你处理。在getopts处理每个选项时，它会将OPTIND环境变量值增一。在getopts完成处理时，你可以使用shift命令和OPTIND值来移动参数

## 将选项标准化

|选项| 描述|
|----|---|
|-a |显示所有对象|
|-c |生成一个计数|
|-d |指定一个目录|
|-e |扩展一个对象|
|-f |指定读入数据的文件|
|-h |显示命令的帮助信息|
|-i |忽略文本大小写|
|-l |产生输出的长格式版本|
|-n |使用非交互模式（批处理）|
|-o |将所有输出重定向到的指定的输出文件|
|-q |以安静模式运行|
|-r |递归地处理目录和文件|
|-s |以安静模式运行|
|-v |生成详细输出|
|-x |排除某个对象|
|-y |对所有问题回答yes|

## 获得用户输入

### 基本的读取

read命令从标准输入（键盘）或另一个文件描述符中接受输入。在收到输入后，read命令会将数据放进一个变量。

read命令包含了-p选项，允许你直接在read命令行指定提示符。

以在read命令行中不指定变量。如果是这样，read命令会将它收到的任何数据都放进特殊环境变量REPLY中

### 超时

read -t选项，指定一个计时器

read -nNum 来统计输入的字符数。当输入的字符达到预设的字符数时，就自动退出，

### 隐藏方式读取

read -s 例如输入密码的时候

### 从文件中读取

每次调用read命令，它都会从文件中读取一行文本。当文件中再没有内容时，read命令会退出并返回非零退出状态码。

# 呈现数据

## 理解输入输出

