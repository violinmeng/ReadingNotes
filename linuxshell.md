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

### 标准文件描述符

linux用文件描述符来标识每个文件对象。

| 文件描述符 | 缩写   | 描述     |
| ---------- | ------ | -------- |
| 0          | STDIN  | 标准输入 |
| 1          | STDOUT | 标准输出 |
| 2          | STDERR | 标准错误 |

#### STDIN

STDIN文件描述符代表shell的标准输入。对终端界面来说，标准输入是键盘。shell从STDIN文件描述符对应的键盘获得输入，在用户输入时处理每个字符。
在使用输入重定向符号（<）时，Linux会用重定向指定的文件来替换标准输入文件描述符。它会读取文件并提取数据，就如同它是键盘上键入的。

#### STDOUT

shell对于错误消息的处理是跟普通输出分开的。如果出现错误信息，这些信息是不会出现在日志文件中

#### STDERR

STDERR文件描述符代表shell的标准错误输出。

### 重定向错误

#### 只重定向错误

```shell
$ ls -al badfile 2> test4 
```

将STDERR文件描述符值放在重定向符号前。该值必须紧紧地放在重定向符号前

#### 重定向错误和数据

```shell
$ ls -al test test2 test3 badtest 2> test6 1> test7 
```

也可以将STDERR和STDOUT的输出重定向到同一个输出文件。为此bash shell提供了特殊的重定向符号&>

## 在脚本中重定向输出

### 临时重定向

```
echo "This is an error message" >&2 #重定向到STDERR
```

默认STDERR会导向STDOUT。

### 永久重定向

用exec命令告诉shell在脚本执行期间重定向某个特定文件描述符。

```shell
exec 1>testout #将STDOUT重定向到文件testout
```

一旦重定向了STDOUT或STDERR，就很难再将它们重定向回原来的位置。

一旦重定向了STDOUT或STDERR，就很难再将它们重定向回原来的位置。

## 在脚本中重定向输入

```shell
exec 0< testfile #将STDIN重定向到文件中，程序将从文件获取输入
```

## 创建自己的重定向

在shell中最多可以有9个打开的文件描述符。

### 创建输出问价描述符

```shell
$ cat test13
#!/bin/bash
# using an alternative file descriptor
exec 3>test13out
echo "This should display on the monitor"
echo "and this should be stored in the file" >&3
echo "Then this should be back on the monitor" 
```

也可以追加到现有文件

```shell
exec 3>>test13out
```

### 重定向问价描述符

```shell
$ cat test14
#!/bin/bash
# storing STDOUT, then coming back to it
exec 3>&1
exec 1>test14out
echo "This should store in the output file"
echo "along with this line."
exec 1>&3
echo "Now things should be back to normal"
$
$ ./test14
Now things should be back to normal
$ cat test14out
This should store in the output file
along with this line.
$
```

可以将一个文件描述符重定向到标准输出，然后重定向标准输出到一个文件，也可以将标准输出重定向到之前的文件描述符。

这是一种在脚本中临时重定向输出，然后回复默认输出设置的常用方法。

### 创建输入文件描述符

在重定向到文件之前，先将STDIN文件描述符保存到另外一个文件描述符，然后在读取完文件之后再将STDIN恢复到它原来的位置。

### 创建读写文件描述符

可以打开单个文件描述符和输出，用同一个文件描述符对同一个文件进行读写。

是对同一个文件进行数据读写，shell会维护一个内部指针，指明在文件中的当前位置。任何读或写都会从文件指针上次的位置开始。

```shell
$ cat test16
#!/bin/bash
# testing input/output file descriptor
exec 3<> testfile
read line <&3
echo "Read: $line"
echo "This is a test line" >&3
$ cat testfile
This is the first line.
This is the second line.
This is the third line.
$ ./test16
Read: This is the first line.
$ cat testfile
This is the first line.
This is a test line
ine.
This is the third line.
$ 
```

### 关闭文件描述符

要关闭文件描述符，将它重定向到特殊符号&-。

```shell
exec 3>&-
```

shell会在脚本退出时自动关闭。

再关闭文件描述符时还需要注意另一件事，如果随后你在脚本中打开了同一个输出文件，shell会用一个新文件来替换已有文件。

## 列出打开的文件描述符

```
/usr/bin/lsof
```

- -p指定进程ID
- -d指定文件描述符编号
- $$当前PID
- -a对其他两个选项进行AND运算。

```shell
$ /usr/sbin/lsof -a -p $$ -d 0,1,2
COMMAND PID USER FD TYPE DEVICE SIZE NODE NAME
bash 3344 rich 0u CHR 136,0 2 /dev/pts/0
bash 3344 rich 1u CHR 136,0 2 /dev/pts/0
bash 3344 rich 2u CHR 136,0 2 /dev/pts/0 
```

## 阻止命令输出

将文件描述符重定向到一个叫作null文件的特殊文件

也可以用来快速清除现有文件中的数据。

```shell
$ cat testfile
This is the first line.
This is the second line.
This is the third line.
$ cat /dev/null > testfile
$ cat testfile
$
```

## 创建临时文件

mktemp命令可以在/tmp目录中创建一个唯一的临时
文件。

它会将文件的读和写权限分配给文件的属主，并将你设成文件的属主。一旦创建了文件，你就在脚本中有了完整的读写权限，但其他人没法访问它（当然，root用户除外）。

### 创建本地临时文件

要用mktemp命令在本地目录中创建一个临时文件，你只要指定一个文件名模板就行了。模板可以包含任意文本文件名，在文件名末尾加上6个X就行了。mktemp命令会用6个字符码替换这6个X，从而保证文件名在目录中是唯一的。

```shell
$ mktemp testing.XXXXXX 
```

### 在/tmp目录中创建临时文件

-t选项会强制mktemp命令来在系统的临时目录来创建该文件。返回用来创建临时文件的全路径。

### 创建临时目录

-d选项告诉mktemp命令来创建一个临时目录而不是临时文件。

```shell
$ mktemp -d dir.XXXXXX
```

## 记录消息

将输出同时发送到显示器和日志文件。可用tee命令。

tee命令相当于管道的一个T型接头。它将从STDIN过来的数据同时发往两处。一处是STDOUT，另一处是tee命令行所指定的文件名：

```shell
tee filename
```

默认情况下，tee命令会在每次使用时覆盖输出文件内容。追加用-a。

## 实例

它读取.csv格式的数据文件，输出SQL INSERT语句来将数据插入数据库

```shell
$cat test23
#!/bin/bash
# read file and create INSERT statements for MySQL
outfile='members.sql'
IFS=','
while read lname fname address city state zip
do
 cat >> $outfile << EOF
 INSERT INTO members (lname,fname,address,city,state,zip) VALUES
('$lname', '$fname', '$address', '$city', '$state', '$zip');
EOF
done < ${1} #读取第一个命令行参数
$
```

# 控制脚本

## 处理信号

### linux信号

默认情况下，bash shell会忽略收到的任何SIGQUIT (3)和SIGTERM (5)信号（正因为这样，交互式shell才不会被意外终止）。

通过SIGINT信号，可以中断shell。Linux内核会停止为shell分配CPU处理时间。这种情况发生时，shell会将SIGINT信号传给所有由它所启动的进程，

### 生成信号

1.终端进程

Ctrl+C会生成SIGINT（终止进程）信号。

2.暂停进程

Ctrl+Z组合键会生成一个SIGTSTP信号，停止shell中运行的任何进程。

停止进程会让程序继续保留在内存中，并能从上次停止的位置继续运行。

用ps -l命令查看进程的时候。

在S列中（进程状态），ps命令将已停止作业的状态为显示为T。这说明命令要么被跟踪，要么被停止了。

kill命令发送SIGKILL信号终止进程。

### 捕获信号

trap命令，如果脚本收到了trap命令中列出的信号，该信号不再由shell处理，而是交由本地处理。

### 捕获脚本退出

只需要在trap命令后加上EXIT信号就行。

### 修改或移除捕获

要想在脚本中的不同位置进行不同的捕获处理，只需重新使用带有新选项的trap命令。

也可以删除已设置好的捕获。只需要在trap命令与希望恢复默认行为的信号列表之间加上两个破折号就行了。

```shell
trap -- SIGINT
```

也可以在trap命令后使用单破折号来恢复信号的默认行为。单破折号和双破折号都可以正常发挥作用。

## 以后台模式运行脚本

### 后台运行脚本

在命令后加个&符号

当后台进程运行时，它仍然会使用终端显示器来显示STDOUT和STDERR消息。

### 运行多个后台作业

每一个后台进程都和终端会话（pts/0）终端联系在一起。如果终端会话退出，那么后台进程也会随之退出。

## 在非控制台下运行脚本

nohup：可以让脚本一直以后台模式运行到结束，即使你退出了终端会话。

nohup命令会自动将STDOUT和STDERR的消息重定向到一个名为nohup.out的文件中。

## 作业控制

作业停止后，可以用kill命令终止该进程，发送SIGCONT信号重启停止的进程。

### 查看作业

| 参  数 | 描  述                                      |
| ------ | ------------------------------------------- |
| -l     | 列出进程的PID以及作业号                     |
| -n     | 只列出上次shell发出的通知后改变了状态的作业 |
| -p     | 只列出作业的PID                             |
| -r     | 只列出运行中的作业                          |
| -s     | 只列出已停止的作业                          |

当前的默认作业完成处理后，带减号的作业成为下一个默认作业。任何时候都只有一个带加号的作业和一个带减号的作业，不管shell中有多少个正在运行的作业。

### 重启停止的作业

可以将已停止的作业作为后台进程或前台进程重启。前台进程会接管你当前工作的终端，所以在使用该功能时要小心了。

以后台模式重启可以用bg命令加上作业号。

以前台模式重启可以用fg命令加上作业号。

## 调整谦让度

linux系统中，由shell启动的所有进程的调度优先级默认都是相同的。

调度优先级是个整数值，从-20（最高优先级）到+19（最低优先级）。默认情况下，bash shell以优先级0来启动所有进程。

### nice命令

-n指定新的优先级级别；（-n并不是必须的）

必须将nice命令和要启动的命令放在同一行中。

普通系统用户不允许提高命令的优先级

### renice命令

改变已经运行命令的优先级。

- 只能对属于你的进程进行renice
- 只能通过renice降低进程的优先级
- root用户可以通过renice任意调整进程的优先级

## 定时运行作业

### 用at命令来计划执行作业

at的守护进程atd会以后台模式运行，atd守护进程会检查系统上的一个特殊目录（通常位于/var/spool/at）来获取用at命令提交的作业。默认情况下，atd守护进程会每60秒检查一下这个目录。

#### 1.at命令的格式

```shell
at [-f filename] time
```

在你使用at命令时，该作业会被提交到作业队列（job queue）。作业队列会保存通过at命令提交的待处理的作业。针对不同优先级，存在26种不同的作业队列。作业队列通常用小写字母a~z和大写字母A~Z来指代。a的优先级最低。

#### 2.获取作业的输出

Linux系统会将提交该作业的用户的电子邮件地址作为STDOUT和STDERR。任何发到STDOUT或STDERR的输出都会通过邮件系统发送给该用户。

#### 3.列出等待的作业

atq命令

#### 4.删除作业

atrm命令

### 定期执行脚本

#### 1.cron时间表

```shell
min hour dayofmonth month dayofweek command

00 12 * * * if [`date +%d -d tomorrow` = 01 ] ; then ; command #每个月最后一天
```

可以用通配符*来指定条目

#### 2.构建cron时间表

ceontab命令

-e可以添加条目

#### 3.浏览cron目录

如果你创建的脚本对精确的执行时间要求不高，用预配置的cron脚本目录会更方便。有4个基本目录：hourly、daily、monthly和weekly。

#### 4.anacron程序

如果anacron知道某个作业错过了执行时间，它会尽快运行该作业。

anacron不会运行位于/etc/cron.hourly的脚本。这是因为anacron程序不会处理执行时间需求小于一天的脚本。

### 使用新shell启动脚本

.bashrc文件通常也是通过某个bash启动文件来运行的。因为.bashrc文件会运行两次：一次是当你登入bash shell时，另一次是当你启动一个bash shell时。



# 创建函数

## 基本的脚本函数

### 创建函数

```shell
function name {
	commands
}

name () {
	COMMANDS
}
```

### 使用函数

如果在函数被定义前使用函数，你会收到一条错误消息。

## 返回值

返回退出状态码

### 默认退出状态码

函数的退出状态码是函数中最后一条命令返回的退出状态码。在函数执行结束后，可以用标准变量$?来确定函数的退出状态码。

### 使用return命令

- 函数一结束就返回值
- 退出状态码必须时0~255

### 使用函数输出

保存到变量中；

```shell
function dbl {    
	read -p "Enter a value: " value    
	echo $[ $value * 2 ] #可返回赋值到变量中
}
```

## 在函数中使用变量

### 向函数传递参数

函数名会在$0变量中定义，函数命令行上的任何参数都会通过$1、$2等定义。也可以用特殊变量$#来判断传给函数的参数数目。

在脚本中指定函数时，必须将参数和函数放在同一行。

由于函数使用特殊参数环境变量作为自己的参数值，因此它无法直接获取脚本在命令行中的参数值。

```shell
$ cat test7 #!/bin/bash 
# trying to access script parameters inside a function 
function func7 {
	echo $[ $1 * $2 ] 
} 
if [ $# -eq 2 ] 
then
	value=$(func7 $1 $2)    
	echo "The result is $value" 
else
	echo "Usage: badtest1 a b" 
fi 
$  
$ ./test7 
Usage: badtest1 a b 
$ ./test7 10 15 
The result is 150
```

### 在函数中处理变量

- 全局变量
- 局部变量

#### 全局变量

全局变量是在shell脚本中任何地方都有效的变量。如果你在脚本的主体部分定义了一个全局变量，那么可以在函数内读取它的值。类似地，如果你在函数内定义了一个全局变量，可以在脚本的主体部分读取它的值。

#### 局部变量

无需在函数中使用全局变量，函数内部使用的任何变量都可以被声明成局部变量。要实现这一点，只要在变量声明的前面加上local关键字就可以了。

## 数组变量和函数

### 向函数传递数组参数

必须将数组变量的值分解成单个的值，然后将这些值作为函数参数使用。在函数内部，可以将所有的参数重新组合成一个新的变量。

```shell
function addarray {
	local sum=0
	local newarray
	newarray=($(echo "$@")) #分解数组单个值，重新组合成新的变量
	for value in ${newarray[*]}
	do
		sum=$[ $sum + $value ] 
	done
	echo $sum
}
```

### 从函数返回数组

从函数里向shell脚本传回数组变量也用类似的方法。函数用echo语句来按正确顺序输出单个数组值，然后脚本再将它们重新放进一个新的数组变量中。

## 函数递归

```shell
function factorial {
	if [ $1 -eq 1 ]
	then
		echo 1
	else
		local temp=$[ $1 - 1 ]
		local result='factorial $temp' 
		echo $[ $result * $1 ]
	fi
}
```

## 创建库

bash shell允许创建函数库文件。

使用函数库的关键在于source命令。source命令会在当前shell上下文中执行命令，而不是创建一个新shell。可以用source命令来在shell脚本中运行库文件脚本。这样脚本就可以使用库中的函数了。

source命令有个快捷的别名，称作点操作符（dot  operator）。要在shell脚本中运行myfuncs库文件，只需添加下面这行：

```shell
. ./myfuncs
```

## 在命令行上使用函数

### 在命令行上创建函数

- 单行方式定义函数，每个命令之后接分号。
- 多行方式定义函数，提示符提示输入更多的命令

在命令行上创建函数时要特别小心。如果你给函数起了个跟内建命令或另一个命令相同的名字，函数将会覆盖原来的命令。

### 在.bashrc文件中定义函数

- 直接定义函数
- 读取函数文件 用source命令将库文件中的函数添加到.bashrc中。

## 实例

### 下载及安装

### 构件库

```shell
.configure
make
make test
make install
```

### shtool库函数

```
shtool [options] [function [options] [args]]
```

### 使用库

# 图形化桌面环境中的脚本编程

## 创建文本菜单

### 创建菜单布局

创建菜单前要用clear命令清空显示器上的已有内容。然后用echo命令显示菜单元素。

最后用read命令读取用户输入。

### 创建菜单函数

为没有实现的函数先创建一个桩函数。针对每个case创建一个函数。

### 添加菜单逻辑

case命令实现，将菜单布局和函数结合起来。

### 整合shell脚本菜单

将以上各个部分组合起来。使用while循环来一直打开菜单。

### 使用select命令

select命令只需要一条命令就可以创建出菜单，然后获取输入的答案并自动处理。

```shell
select variable in list 
do
	case commands
done
```

select语句中的所有内容必须作为一行出现。

## 制作窗口

### dialog包

dialog命令使用命令行参数来决定生成哪种窗口部件（widget）。

```shell
dialog --widget parameters #parameters定义了部件窗口的大小以及部件需要的文本
```

每个dialog部件都提供两种形式的输出

- STDERR，部件返回数据，将STDERR输出重定向到另一个文件或文件描述符中。
- 使用退出状态码，确定用户选择的按钮

#### msgbox部件

```shell
dialog --msgbox text height width
```

#### yesno部件

dialog命令的退出状态码会根据用户选择的按钮来设置

#### inputbox部件

```shell
dialog --inputbox "Enter your age:" 10 20 2>age.txt
```

#### textbox部件

#### menu部件

```shell
$ dialog --menu "Sys Admin Menu" 20 30 10 1 "Display disk space" 2 "Display users" 3 "Display memory usage" 4 "Exit" 2> test.txt
```

#### fselect部件

```shell
$ dialog --title "Select a file" --fselect $HOME/ 10 50 2>file.txt
```

### dialog选项

config部件外观和配置。

--backtitle选项是为脚本中的菜单创建公共标题的简便办法。

### 在脚本中使用dialog命令

## 使用图形

### kde环境

kdialog命令

```shell
kdialog display-options window-options arguments 
```

kdialog窗口部件用STDOUT来输出值，而不是STDERR。

### GNOME环境

- gdialog
- zenity

#### zenity

输出也是STDOUT

# 初始sed和gawk

## 文本处理

### sed编辑器

流编辑器

- 一次从输入值读取一行数据
- 根据所提供的编辑器命令匹配数据
- 按照命令修改流中的数据
- 将新的数据输入到STDOUT

```shell
sed options script file	
```

s命令，替换字符串

sed编辑器并不会修改文本文件的数据。它只会将修改后的数据发送到STDOUT。

使用多个编辑命令使用-e选项，命令之间用分号隔开。也可以用次提示符来分隔命令（‘）

从文件中读取编辑器命令： -f选项指定文件

### gawk程序

能提供一个类编程环境来修改和重新组织文件中的数据。

- 定义变量来保存数据
- 使用算术和字符串操作符来处理数据
- 使用结构化的编程概念（比如if-then语句和循环）来为数据处理增加处理逻辑
- 通过提取数据文件中的数据元素，将其重新排列或格式化，生产格式化报告。

```shell
gawk options program file
```

gawk程序用一对花括号来定义。

```shell
$ gawk '{print "Hello World!"}'
```

Ctrl+D组合键会在bash中产生一个EOF字符。这个组合键能够终止该gawk程序并返回到命令行界面提示符下。

- $0代表整个文本行
- $1代表文本行中的第一个数据字段
- $2代表文本行中的第二个数据字段
- $n代表文本行中的第n个数据字段

每个数据字段通过字段分隔符划分。

| 选  项       | 描  述                             |
| ------------ | ---------------------------------- |
| -F fs        | 指定行中划分数据字段的字段分隔符   |
| -f file      | 从指定的文件中读取程序             |
| -v var=value | 定义gawk程序中的一个变量及其默认值 |
| -mf N        | 指定要处理的数据文件中的最大字段数 |
| -mr N        | 指定数据文件中的最大数据行数       |
| -W keyword   | 指定gawk的兼容模式或警告等级       |

在多个命令之间放置分号。

注意，gawk程序在引用变量值时并未像shell脚本一样使用美元符。

在处理数据前处理脚本，比如为报告创建标题，BEGIN关键字针对这种情况。

在处理数据后运行脚本，用END

## sed编辑器基础

### 更多的替换选项

- 替换标记 要让替换命令能够替换一行中不同地方出现的文本必须使用替换标记（substitution flag）。替换标记会在替换命令字符串之后设置。
  - 数字，表明新文本将替换第几处模式匹配的地方
  - g，表明新文本将会替换所有匹配的文本
  - p，表明原先行的内容要打印处理 和-n同时使用打印匹配的行
  - w file，将替换结果写道文件中
- 替换字符 替换/字符是需要转义，或者使用其他字符来作为替换命令中的字符串分隔符。

### 使用地址

- 以数字形式表示行区间 行号从1开始
- 用文本模式来过滤出行：指定文本模式滤除命令要作用的行

```shell
[address] command

address {
	command1
	command2
	command3
}#将地址的多个命令分组

/pattern/command #文本模式过滤，支持正则表达式
```

$表示最后一行

可以用花括号将多条命令组合在一起，作用在同一行上。

### 删除行

命令d，删除匹配指定寻址模式的所有行，支持寻址模式和文本过滤模式。

可以使用两个文本模式来删除某个区间内的行，sed编辑器会删除两个指定行之间的所有行（包括指定的行）。

如果第一个匹配，第二个不匹配，会把匹配之后的所有行删除。

### 插入和附加文本

- 插入命令（i）会在指定行之前增加一个新行
- 附加命令（a）会在指定行之后增加一个新行

```shell
sed '[address] command \
new line'
```

可以在用这些命令时只指定一个行地址。可以匹配一个数字行号或文本模式，但不能用地址区间。

### 修改行

命令c，可以在用这些命令时只指定一个行地址。可以匹配一个数字行号或文本模式，使用区间会替换区间内的所有文本。

### 转换命令（transform）

```shell
[address]y/inchars/outchars
```

如果inchars和outchars的长度不同，则sed编辑器会产生一条错误消息。

转换命令是一个全局命令，也就是说，它会文本行中找到的所有指定字符自动进行转换，而不会考虑它们出现的位置。

### 回顾打印

- p命令用来打印文本行

  - ```shell
    sed -n '/number 3/p' data.txt #打印修改行
    sed -n '2,3p' data.txt #打印2，3行
    
    $ sed -n '/3/{ 
    > p 
    > s/line/test/p 
    > }' data6.txt #打印修改前后的行
    ```

- 等号（=）命令用来打印行号：打印当前行号。

  - ```shell
    $ sed -n '/number 4/{ 
    > = 
    > p 
    > }' data6.txt 4 This is line number 4. 
    ```

- l命令用来列出行（list）：打印数据流中的文本和不可打印的ASCII字符。任何不可打印字符要么在其八进制值前加一个反斜线，要么使用标准C风格的命名法。

### 使用sed处理文件

- 写入文件。

  ```shell
  [address]w filename
  
  sed -n '/Browncoat/w Browncoats.txt' data11.txt 
  ```

- 从文件读取数据

  ```shell
  [address]r filename #将读取的文件插入到指定地址后
  
  sed '3r data12.txt' data6.txt
  ```

  读取命令的另一个很酷的用法是和删除命令配合使用：利用另一个文件中的数据来替换文件中的占位文本。

# 正则表达式

## 定义

正则表达式是你所定义的模式模板（pattern temple），可以用它来过滤文本。

## 类型

不同应用程序会用不同类型的正则表达式。包括编程语言，linux使用工具，以及主流应用（mysql，postgresql）

正则表达式是通过**正则表达式引擎**（负责解释正则表达式模式，使用模式进行文本匹配）实现的。

两种流行的正则表达式引擎

- POSIX基础正则表达式（basic regular expression，BRE）引擎
- POSIX扩展正则表达式（extended regular expression，ERE）引擎

## 定义BRE模式

### 纯文本

```shell
$ echo "This is a test" | sed -n '/this/p'　#区分大小写
```

### 特殊字符

```shell
.*[]^${}\+?|()
```

文本中使用这些字符需要转义

/（正斜线）也需要转义

### 锚字符

默认情况下，当指定一个正则表达式模式时，只要模式出现在数据流中的任何地方，它就能匹配。有两个特殊字符可以用来将模式锁定在数据流中的行首或行尾。

- 锁定在行首（^（脱字符））

  如果你将脱字符放到模式开头之外的其他位置，那么它就跟普通字符一样，不再是特殊字符了。

  如果指定正则表达式模式时只用了脱字符，就不需要用反斜线来转义。但如果你在模式中先指定了脱字符，随后还有其他一些文本，那么你必须在脱字符前用转义字符。

- 锁定在行尾（$）

- 组合锚点

  查找含特定文本的数据行

  将两个锚点直接组合在一起，之间不加任何文本，这样过滤出数据流中的空白行。

### 点号字符

匹配除换行符之外的任意字符。必须匹配一个字符。

### 字符组（character class）

用`[]`定义。

字符组的一个极其常见的用法是解析拼错的单词，比如用户表单输入的数据。

### 排除型字符组

用`[^]`定义

### 区间

指定区间的第一个字符、单破折线以及区间的最后一个字符。

### 特殊的字符组

| 组          | 描  述                                         |
| ----------- | ---------------------------------------------- |
| [[:alpha:]] | 匹配任意字母字符，不管是大写还是小写           |
| [[:alnum:]] | 匹配任意字母数字字符0~9、A~Z或a~z              |
| [[:blank:]] | 匹配空格或制表符                               |
| [[:digit:]] | 匹配0~9之间的数字                              |
| [[:lower:]] | 匹配小写字母字符a~z                            |
| [[:print:]] | 匹配任意可打印字符                             |
| [[:punct:]] | 匹配标点符号                                   |
| [[:space:]] | 匹配任意空白字符：空格、制表符、NL、FF、VT和CR |
| [[:upper:]] | 匹配任意大写字母字符A~Z                        |

### 星号

在字符后面放置星号表明该字符必须在匹配模式的文本中出现0次或多次。

## 扩展正则表达式

gawk支持ERE，awk不支持

### 问号

问号表明前面的字符可以出现0次或1次

### 加号

加号表明前面的字符可以出现1次或多次，但必须至少出现1次。

### 花括号

ERE中的花括号允许你为可重复的正则表达式指定一个上限。这通常称为间隔（interval）。

两种形式

- m：正则表达式准确出现n次
- m，n：正则表达式至少出现m次，至多出现n次。

默认情况下，gawk程序不会识别正则表达式间隔。必须指定gawk程序的--re- interval命令行选项才能识别正则表达式间隔。

### 管道符号

管道符号允许你在检查数据流时，用逻辑OR方式指定正则表达式引擎要用的两个或多个模式。

```shell
expr1|expr2|...
```

### 表达式分组

正则表达式模式也可以用圆括号进行分组。当你将正则表达式模式分组时，该组会被视为一个标准字符。可以像对普通字符一样给该组使用特殊字符。

## 实战

### 目录文件计数

```powershell
$ cat countfiles 
#!/bin/bash
# count number of files in your PATH 
mypath=$(echo $PATH | sed 's/:/ /g')
count=0
for directory in $mypath
do    
	check=$(ls $directory) 
    for item in $check
    do 
    	count=$[ $count + 1 ]
    done
    echo "$directory - $count"
    count=0
done
$ ./countfiles
/usr/local/sbin - 0
/usr/local/bin - 2 
/usr/sbin - 213 
/usr/bin - 1427 
/sbin - 186 
/bin - 152 
/usr/games - 5 
```

### 验证电话号码

```shell
(123)456-7890
(123) 456-7890
123-456-7890
123.456.7890

^\(?[2-9][0-9]{2}\)?(| |-|\.)[0-9]{3}( |-|\.)[0-9]{4}$
```

### 解析邮件地址

```shell
^([a-zA-Z0-9_\-\.\+]+)@([a-zA-Z0-9_\-\.]+)\.([a-zA-Z]{2,5})$
```

# sed进阶

## 多行命令

- N：将数据流中的下一行加起来创建一个多行组来处理
- D：删除多行组中的一行
- P：打印多行组中的一行

### next命令





### 分支

可以为分支命令定义一个跳转的标签，标签以冒号开始，最多可以是7个标签长度。

```shell
：label
```

如果某行匹配了分支模式，  sed编辑器就会跳转到带有分支标签的那行。

### 测试

测试（test）命令（t）也可以用来改变sed编辑器脚本的执行流程。测试命令会根据替换命令的结果跳转到某个标签，而不是根据地址进行跳转。

```shell
[address]t [label]
```

测试命令提供了对数据流中的文本执行基本的if-then语句的一个低成本办法。

```shell
$ echo "This, is, a, test, to, remove, commas. " | sed -n '{
> :start
> s/,//1p
> t start
> }' 
This is, a, test, to, remove, commas. 
This is a, test, to, remove, commas. 
This is a test, to, remove, commas.
This is a test to, remove, commas.
This is a test to remove, commas.
This is a test to remove commas. #当无需替换时，测试命令不会跳转而是继续执行剩下的脚本。
```

## 模式替代

在使用通配符时，很难知道到底哪些文本会匹配模式。

### &符号

&符号可以用来代表替换命令中的匹配的模式。

### 替代单独的单词

sed编辑器用圆括号来定义替换模式中的子模式。你可以在替代模式中使用特殊字符来引用每个子模式。替代字符由反斜线和数字组成。数字表明子模式的位置。sed编辑器会给第一个子模式分配字符\1，给第二个子模式分配字符\2，依此类推。

```sh
$ echo "The System Administrator manual" | sed ' > s/\(System\) Administrator/\1 User/' 
The System User manual $
```



```sh
$ echo "1234567" | sed '{ 
> :start > s/\(.*[0-9]\)\([0-9]\{3\}\)/\1,\2/ 
> t start > }' 
1,234,567 
$
```

## 在脚本中使用sed

### 使用包装脚本

可以将sed编辑器命令放到shell包装脚本（wrapper）中，不用每次使用时都重新键入整个脚本。

### 重定向sed的输出

可以在脚本中用$()将sed编辑器命令的输出重定向到一个变量中，以备后用。

## 创建sed实用工具

### 加倍行间距

每行之后插入空白行

```sh
sed 'G' data2.txt #G命令会简单地将保持空间内容附加到模式空间内容后。当启动sed编辑器时，保持空间只有一个空行。将它附加到已有行后面，你就在已有行后面创建了一个空白行。最后一行也有一个空白
sed '$!G' data2.txt #改进后，最后一行没有空白了。
```

### 对可能含有空白行的文件夹背行间距

```sh
$ sed '/^$/d ; $!G' data6.txt #先删除已有空白行
```

### 给文件中的行编号 

```sh
$ sed '=' data2.txt | sed 'N; s/\n/ /'#也可以用命令nl
```

### 打印末尾行

```sh
$ sed '{
> :start
> $q ; N ; 11,$D #q退出命令，判断是不是最后一行 D命令会删除模式空间的第一行
> b start
> }' data7.txt 
```

### 删除行

- 删除连续的空白行

  ```sh
  sed '/./,/^$/!d' data8.txt #区间是/./到/^$/，其余的空白行都删除
  ```

- 删除结尾的空白行

  ```sh
  sed '{ 
  :start 
  /^\n*$/{$d; N; b start } 
  }'
  ```

  