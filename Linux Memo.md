# 1.Linux11

## 1.Base

### 1.1 常用命令

#### 1.1.1 新建文件

```bash
touch [文件名]  如果文件不存在，新建文件（如果存在，修改日期）
```

#### 1.1.2 创建目录

```bash
mkdir [目录名]  
-p   可以递归创建目录（例：mkdir -p 1/2/3/4/5/6) 
```

#### 1.1.3 删除指定的文件名

```bash
rm [文件名] (注：不能恢复）
-r 	       递归删除目录下的内容/删除文件夹
-f         强行删除，如果没有不会提实
```

#### 1.1.4 查看ip地址

```bash
hostname -I
```

#### 1.1.5 切换图形界面

```bash
runlevel  现在使用的界面级别
init 3    change CLI
init 5	  change GUI
```

#### 1.1.6 登陆时&前的提示

```bash
/etc/motd     时
/etc/issue	  前
```

#### 1.1.7 显示当前使用的shell

```bash
echo ${SHELL}
cat /etc/shells   查看正在使用的系统shlle
```

**shell中可执行的两类命令**

- 内部命令：由shell自带的，而且通过某命令形式提供，用户登录后自动加载并常驻内存中
- 外部命令：在文件系统路径下有对应的可执行程序文件，当执行命令时才从磁盘加载至内存中，执行完毕后从内存中删除

**区别指定的命令时内部或外部命令**

```bash
type [COMMAND]
```

**范例**：**查看是否存在对应内部和外部命令**

```bash
[root@rocky84 ~]#type -a echo
echo is a shell builtin
echo is /usr/bin/echo
```

#### 1.1.8 当前运行的所以程序

```bash
ps aux
```

#### 1.1.9  更改字体颜色

```bash
(CentOS)echo 'PS1="\[\e[1;32m\][\t \[\e[1;33m\]\u\[\e[35m\]@\h\[\e[1;31m\] \W\[\e[1;32m\]]\[\e[0m\]\\$"' > /etc/profile.d/env.sh
```

#### 1.1.10 更改hostname

```bash
root@test:~# hostnamectl set-hostname [ubuntu20-server]
```

#### 1.1.11 命令别名

对于经常执行的较长的命令，可以将其定义成较短的别名，以方便执行 

```bash
alias   显示当前shell进程所有可用的命令别名
alias [NAME]=[VALUE]  定义别名NAME，其相对于执行命令VALUE
```

##### 1.1.11.1 撤销别名：unalias

```bash
unalias [-a] name [name ...]
unalias -a   #取消所有别名
注意：在命令行中定义的别名，仅对当前shell进程有效
如果想永久有效，要定义在配置文件中
	• 仅对当前用户: ~/.bashrc
    • 对所有用户有效：/etc/bashrc
编辑配置给出的新配置不会立即生效，bash进程重新读取配置文件
source /path/to/config_file
. /path/to/config_file
```

**如果别名同原命令同名，如果要执行原命令，可使用**

```bash
\ALIASNAME
"ALIASNAME"
'ALIASNAME'
command ALIASNAME
/path/command #只适用于外部命令
```

##### 1.1.11.2 命令格式

```bash
COMMAND [OPTIONS...] [ARGUMENTS..]
COMMAND [COMMAND] [COMMAND] ...
```

选项：用于启动或关闭命令的某个或某些功能

短选项：UNIX 风格选项，-c 例如：-l,-h
长选项：GNU 风格选项，–word 例如：–all,–human
BSD风格选项：一个字母，例如：a,使用相对较少
参数：命令的作用对象，比如：文件名，用户名等

**注意**：

多个选项以及多参数和命令之间使用空白符分隔
取消和结束命令执行：Ctrl+c, Ctrl+d
多个命令可以用";"符号分开
一个命令可以用\分成多行

#### 1.1.12 从新识别硬盘

```bash
echo - - - > /sys/class/scsi_host/host0/scan;echo - - - > /sys/class/scsi_host/host1/scan;echo - - - > /sys/class/scsi_host/host2/scan
```

### 1.2 查看相关命令

#### 1.2.1 查看CPU

```bash
lscpu 
cat /proc/cpuinfo  
```

#### 1.2.2 查看内存大小

```bash
free
cat /proc/meminfo
```

#### 1.2.3 查看硬盘和分区情况

```bash
lsblk
cat /proc/partitions 
```

#### 1.2.4 查看系统版本信息

```bash
arch 查看系统架构
uname -r 查看内核版本
cat /etc/redhat-release  查看操作系统发行版本（红帽系列centOS）
cat /etc/os-release    查看操作系统发行版本（Ubuntu）
lsb_release -a     查看发行版本（ubuntu）
```

#### 1.2.5 日期和时间

Linux的两种时钟

系统时钟：有Linux内核通过CPU的工作频率进行的
硬件时钟：主板
相关命令
date 显示和设置系统时间

```bash
date  显示时间
clock，hwclock: 显示硬件时钟
-s, --hctosys #以硬件时钟为准，校正系统时钟
-w, --systohc #以系统时钟为准，校正硬件时钟
```

```bash
用法：date [选项]... [+格式]
　或：date [-u|--utc|--universal] [MMDDhhmm[[CC]YY][.ss]]
以给定的格式显示当前时间，或是设置系统日期。
 
  -d,--date=字符串              显示指定字符串所描述的时间，而非当前时间
  -f,--file=日期文件            类似--date，从日期文件中按行读入时间描述
  -r, --reference=文件          显示文件指定文件的最后修改时间
  -R, --rfc-2822                以RFC 2822格式输出日期和时间
                                例如：2006年8月7日，星期一 12:34:56 -0600
      --rfc-3339=TIMESPEC       以RFC 3339 格式输出日期和时间。
                                TIMESPEC=`date'，`seconds'，或 `ns' 
                                表示日期和时间的显示精度。
                                日期和时间单元由单个的空格分开：
                                2006-08-07 12:34:56-06:00
  -s, --set=字符串              设置指定字符串来分开时间
  -u, --utc, --universal        输出或者设置协调的通用时间
      --help            显示此帮助信息并退出
      --version         显示版本信息并退出
 
给定的格式FORMAT 控制着输出，解释序列如下：
 
  %%    一个文字的 %
  %a    当前locale 的星期名缩写(例如： 日，代表星期日)
  %A    当前locale 的星期名全称 (如：星期日)
  %b    当前locale 的月名缩写 (如：一，代表一月)
  %B    当前locale 的月名全称 (如：一月)
  %c    当前locale 的日期和时间 (如：2005年3月3日 星期四 23:05:25)
  %C    世纪；比如 %Y，通常为省略当前年份的后两位数字(例如：20)
  %d    按月计的日期(例如：01)
  %D    按月计的日期；等于%m/%d/%y
  %e    按月计的日期，添加空格，等于%_d
  %F    完整日期格式，等价于 %Y-%m-%d
  %g    ISO-8601 格式年份的最后两位 (参见%G)
  %G    ISO-8601 格式年份 (参见%V)，一般只和 %V 结合使用
  %h    等于%b
  %H    小时(00-23)
  %I    小时(00-12)
  %j    按年计的日期(001-366)
  %k    时(0-23)
  %l    时(1-12)
  %m    月份(01-12)
  %M    分(00-59)
  %n    换行
  %N    纳秒(000000000-999999999)
  %p    当前locale 下的"上午"或者"下午"，未知时输出为空
  %P    与%p 类似，但是输出小写字母
  %r    当前locale 下的 12 小时时钟时间 (如：11:11:04 下午)
  %R    24 小时时间的时和分，等价于 %H:%M
  %s    自UTC 时间 1970-01-01 00:00:00 以来所经过的秒数
  %S    秒(00-60)
  %t    输出制表符 Tab
  %T    时间，等于%H:%M:%S
  %u    星期，1 代表星期一
  %U    一年中的第几周，以周日为每星期第一天(00-53)
  %V    ISO-8601 格式规范下的一年中第几周，以周一为每星期第一天(01-53)
  %w    一星期中的第几日(0-6)，0 代表周一
  %W    一年中的第几周，以周一为每星期第一天(00-53)
  %x    当前locale 下的日期描述 (如：12/31/99)
  %X    当前locale 下的时间描述 (如：23:13:48)
  %y    年份最后两位数位 (00-99)
  %Y    年份
  %z +hhmm              数字时区(例如，-0400)
  %:z +hh:mm            数字时区(例如，-04:00)
  %::z +hh:mm:ss        数字时区(例如，-04:00:00)
  %:::z                 数字时区带有必要的精度 (例如，-04，+05:30)
  %Z                    按字母表排序的时区缩写 (例如，EDT)
 
默认情况下，日期的数字区域以0 填充。
以下可选标记可以跟在"%"后:
 
  - (连字符)不填充该域
  _ (下划线)以空格填充
  0 (数字0)以0 填充
  ^ 如果可能，使用大写字母
  # 如果可能，使用相反的大小写
 
在任何标记之后还允许一个可选的域宽度指定，它是一个十进制数字。
作为一个可选的修饰声明，它可以是E，在可能的情况下使用本地环境关联的
表示方式；或者是O，在可能的情况下使用本地环境关联的数字符号。
```



##### 1.2.5.1 时区

```bash
/etc/localtime   查看是时区

[root@rocky84 ~]##timedatectl list-timezones 
[root@rocky84 ~]#timedatectl list-timezones Asia/Seoul
[root@rocky84 ~]#timedatectl status
cal -y  显示日历  例：#cal 9 1752
```

#### 1.2.6 关机和重启

关机：

```bash
halt
powerOff
```

重启：

```bash
reboot
  -f: 强制，不调用shutdown
  -p: 切断电源
Ctrl+alt+delete 三个键
```

关机或重启：shutdown

```bash
shutdown [OPTIONS...] [TIME] [WALL...]
  -r：reboot    
  -h：halt
  -c：Cancel 
     TIME:无指定，默认相对于+1（Centos7）
     now:立即，相对于+0
     +#：相对时间表示法，几分钟之后：例如 +3
     hh:mm: 绝对时间表示，指明具体时间
```

#### 1.2.7 用户登录信息查看命令

```bash
whoami: 显示当前登录有效用户
who: 系统当前所有的登录会话
w: 系统当前所有的登录会话及所做的操作
```

### 1.3 会话管理

命令行的典型使用方式是，打开一个终端窗口（terminal window，一下简称“窗口”），在里面输入命令。用户与计算机的这种临时交互，称为一次“会话”（session）

会话的一个重要特点是，窗口与其中启动的进程是连在一起的。打开窗口，会话开始；关闭窗口，会话结束，会话内部的进程也会随之终止，不管有没有运行完

一个典型的例子就是，SSH登录远程计算机，打开一个远程窗口执行命令。这时，网络突然断线，再次登录的时候，是找不回上一次执行的命令的。因为上一次ssh会话以及终止了，里面的进程也随之消失了。为了解决这个问题，会话与窗口可以“解绑”：窗口关闭时，会话并不终止，而是继续运行，而是继续运行，等到以后需要的时候，再让会话“绑定”其他窗口

终端复用器软件就是会话与窗口的“解绑”工具，将同名彻底分离。
（1）它允许在单个窗口中，同时访问多个会话。这对于同时运行多个命令行进程很有用。
（2）它可以让窗口“接入”已经存在的会话。
（3）它允许每个会话有多个连接窗口，因此可以多人实时共享会话。
（4）它还支持窗口任意的垂直和水平拆分。

类似的终端复用器还有screen，Tmux

#### 1.3.1  screen

利用screen可以实现会话管理，如：新建会话，共享会话等
注意：CentOS7 来源于base源，CentOS8 来自于epel源

```bash
#centos8 安装screen
[root@centos84 ~]# dnf -y install epel-release
[root@centos84 ~]# dnf -y install screen
```

screen命令常见用法：

```bash
screen -s [SESSION] 创建新screen会话
screen -s [SESSION] 加入screen会话
exit 退出并关闭screen会话
Ctrl+a,d 剥离当前screen会话
screen -ls 显示所有已经打开的screen会话
恢复某screen会话 恢复某screen会话
```

#### 1.3.2 tmux

Tmux 是一个终端复用器（terminal multiplexer）,类似screen，但是更易用，也跟强大

Tmux 就是会话与窗口的“解绑”工具，将它们彻底分离，功能如下

- 它允许在单个窗口中，同时访问多个会话。这对于同时运行多个命令行程序很有用。
- 它可以让新窗口“接入”已经存在的会话。
- 它允许每个会话有多个连接窗口，因此可以多人实时共享会话。
- 它还支持窗口任意的垂直和水平拆分

```bash
yum install tmux 安装
[root@centos84 ~]# tmux  启动
[root@centos84 ~]# exit  退出
logout
```

Tmux窗口有大量的快捷键。所有快捷键都要通过前缀唤起。默认的前缀键是Ctrl+b，即先按下Ctrl+b，快捷键才会生效。帮助命令的快捷键是Ctrl+b 然后，按下 q 键，就可以退出帮助

**新建会话**
第一个启动的 Tmux 窗口，编号为0，第二个窗口编号是1，以此类推。这些窗口对应的会话，就是0号会话、1号会话。使用编号区分会话，不太直观，更好的方法是为会话起名。下面命令新建一个指定的会话。

```bash
tmux new -s <session-name>
```

tmux ls 或 Ctrl+b,s 可以查看当前所有的Tmux会话

```bash
tmux ls
tmux list-session
```

分离会话
在Tmux窗口中，按下 Ctrl+b,d或者输入tmux detach命令，就会将当前会话与窗口分离。

```bash
tmux detach
```

接入会话
tmux attach 命令用于重新接入某个已存在的会话

```bash
tmux attach -t <session-name>   例：tmux attach -t 0
```

杀死会话
tmux kill-sessiony命令用于杀死某个会话。

```bash
tmux kill-session -t <session-name>
```

切换会话
tmux switch命令用于切换会话

```bash
tmux switch -t <session-name>
```

可以将窗口分成多个窗格（pane），每个窗口运行不同的命令

**上下分离格**

```bash
tmux split-window
Ctrl+b,"
```

**左右分离格**

```bash
tmux split-window -h
Ctrl+b,%
```

**窗格快捷键**

```bash
Ctrl+b %：划分左右两个窗格
Ctrl+b "：划分上下两个窗格
Ctrl+b <arrow key>: 光标切换到其他窗格。<arrow key>是指向要切换到窗格的方向键，比如切换到下方窗格，就按方向键 
Ctrl+b ;:光标切换到上一个窗格
Ctrl+b o: 光标切换到下一个窗格
Ctrl+b {：当前窗格左移
Ctrl+b }：当前窗格左移
Ctrl+b Ctrl+o：当前窗格上移
Ctrl+b Alt+o：当前窗格下移
Ctrl+b x：关闭当前窗格
Ctrl+b !: 当前窗格窗格拆分为一个独立窗口
Ctrl+b z: 当前窗格全屏显示，再使用一次变回原来大小
Ctrl+b Ctrl+<arrow key>: 按箭头方向调整窗格大小
Ctrl+b q：显示窗格编号
```

窗口管理

除了将一个窗口划分成多个窗格，Tmux也允许新建多个窗口
新建窗口
tmux new-window命令用来创建新窗口

```bash
tmux new-window
```

新建一个指定名称的窗口

```bash
tmux new-window -n <window-name>
```

切换窗口
tmux select-window命令用来切换窗口
切换到指定编号的窗口

```bash
tmux select-window -t <window-number>
```

切换到指定名称的窗口

```bash
tmux select-window -t <window-name>
```

窗口快捷键

```bash
Ctrl+b c：创建一个新窗口，状态栏会显示多个窗口的信息。
Ctrl+b &：删除当前窗口
Ctrl+b p：切换到上一个窗口（按照状态栏上的顺序）。
Ctrl+b n：切换到下一个窗口。
Ctrl+b l：前后两个窗口来回切换
Ctrl+b <number>：切换到指定编号的窗口，其中的<code><number>是状态栏上的窗口编号。
Ctrl+b w：从列表中选择窗口。
Ctrl+b ,：窗口重命名。
Ctrl+b f： 在窗口列表中招
```

列出所有快捷键，及其对应的tmux命令

```bash
tmux list-keys
```

例如所有Tmux命令及其参数

```bash
tmux list-commands
```

#### 1.3.3 输出信息 echo

**基本用法**

echo 命令可以将后面跟的字符进行输出
功能：显示字符，echo会将输入的字符串送往标准输出。输出的字符串间以空白符隔开，并在最后加上换行号
语法：

```bash
echo [-neE] [字符串]
```

选项：

- -E （默认）不支持\解释功能
- -n 不自动换行
- -e 启用 \ 字符的解释功能

显示变量

```bash
echo "$VAR_NAME" #用变量值替换，弱引用
echo "$VAR_NAME" #变量不会替换，强引用
```

启动命令选项-e,若字符串中出现以下字符，则特别加以处理，而不会将它当成一般文字输出

- \a 发出警告声
- \b 退格键
- \c 最后不加上换行号
- \e escape，相对于\033
- \n 换行且光标移至行首
- \r回车，即光标移至行首，但不换行
- \t 插入tab
- \ 插入\字符
- \0nnn 插入nnn (八进制)所代表的ASCII字符
- \xHH插入HH（十六进制）所代表的ASCII数字（man 7 ascii）

范例：

```bash
[root@centos84 ~]# echo -e 'a\x0Ab'
a
b
[root@centos84 ~]# echo -e '\033[43;31;1;5mmagedu\e[0m'
magedu
[root@centos84 ~]# echo -e '\x57\x41\x4E\x47'
WANG
[root@centos84 ~]# echo \$PATH
$PATH
[root@centos84 ~]# echo \
> ^C
[root@centos84 ~]# echo \\
\
[root@centos84 ~]# echo \\\
> ^C
[root@centos84 ~]# echo \\\\
\\
[root@centos84 ~]# echo "$PATH"
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
[root@centos84 ~]# echo '$PATH'
$PATH
```

**echo** **高级用法**

在终端中，ANSI定义了用于屏幕显示的Escape屏幕控制码
可以显示具有颜色的字符，其格式如下：

```bash
"\033[字体背景颜色;字体颜色m字符串\033[0m"
```

\033[30m – \033[37m 设置前景色
\033[40m – \033[47m 设置背景色

```bash
#字体背景颜色范围: 40--47
40：黑
41：红
42：绿
43：黄
44：蓝
45：紫
46：深绿
47：白色

#字体颜色：30--37
30：黑
31：红
32：绿
33：黄
34：蓝
35：紫
36：深绿
37：白色
```

加颜色只是以下控制码中的一种，下面是常见的一些ANSI控制码

```bash
\033[0m   关闭所有属性
\033[1m   设置高亮度
\033[4m   下划线
\033[5m    闪烁
\033[7m    反显
\033[8m    消隐
\033[nA    光标上移n行
\033[nB    光标下移n行
\033[nC    光标右移n列
\033[nD    光标左移n列
\033[x;yH  设置光标位置x行y列
\033[2J     清屏
\033[K      清除从光标到行尾的内容 
\033[s      保存光标位置    
\033[u      恢复光标位置号
\033[?25l   隐藏光标
\033[?25h   显示光标
\033[2J\033[0;0H  清屏且将光标置顶
```

#### 1.3.4 字符串和编码

查看ASCII表

```bash
[root@centos84 ~]# dnf -y install man-pages
[root@centos84 ~]# man ascii
```

#### 1.3.5 命令行扩展和被括起来的集合

##### 1.3.5.1 **命令行扩展：``和$()**

把一个命令的输出打印给另一个命令的参数，放在``中的一定是有输出信息的命令

```bash
$(COMMAND) 或 `COMMAND`           `` == $()
```

范例：比较"",‘’,``三者区别

```bash
test@ubuntu20-server:~$ echo "echo $HOSTNAME"
echo ubuntu20-server
test@ubuntu20-server:~$ echo 'echo $HOSTNAME'
echo $HOSTNAME
test@ubuntu20-server:~$ echo `echo $HOSTNAME`
ubuntu20-server
#结论
单引号：强引用，六亲不认，变量和命令都不识别，都当成了普通的字符串，“最傻”
双引号：弱引用，不能识别命令，可以识别变量，“半傻不精”
反向单引号：里面的内容必须是能执行的命令并且有输出信息，变量和命令都识别，并且会将反向单引号的内容当成命令进行执行后，再交给调用反向单引号的命令继续，“最聪明”
```

```bash
[root@centos84 ~]# echo "This system's name is $(hostname)"
This system's name is centos84
[root@centos84 ~]# echo "I am `whoami`"
I am root
[root@centos84 ~]# touch $(date +%F).log
[root@centos84 ~]# ls 
2021-09-18.log  anaconda-ks.cfg  motd
[root@centos84 ~]# ll
total 8
-rw-r--r--  1 root root    0 Sep 18 15:04 2021-09-18.log
-rw-------. 1 root root 1258 Sep 17 16:34 anaconda-ks.cfg
-rw-r--r--. 1 root root  587 Aug 22  2020 motd
[root@centos84 ~]# touch `date +%F`.txt
[root@centos84 ~]# ll
total 8
-rw-r--r--  1 root root    0 Sep 18 15:04 2021-09-18.log
-rw-r--r--  1 root root    0 Sep 18 15:05 2021-09-18.txt
-rw-------. 1 root root 1258 Sep 17 16:34 anaconda-ks.cfg
-rw-r--r--. 1 root root  587 Aug 22  2020 motd
[root@centos84 ~]# touch `hostname`-`date +%F`.log
[root@centos84 ~]# ll
total 8
-rw-r--r--  1 root root    0 Sep 18 15:04 2021-09-18.log
-rw-r--r--  1 root root    0 Sep 18 15:05 2021-09-18.txt
-rw-------. 1 root root 1258 Sep 17 16:34 anaconda-ks.cfg
-rw-r--r--  1 root root    0 Sep 18 15:05 centos84-2021-09-18.log
-rw-r--r--. 1 root root  587 Aug 22  2020 motd
[root@centos84 ~]# touch `date +%F_%H-$M-%S`.log
[root@centos84 ~]# touch `date -d '-1 day' +%F`.log
```

范例：$() 和 ``

```bash
[root@centos84 ~]# ll `echo `date +%F`.txt`
-bash: .txt: command not found
ls: cannot access 'date': No such file or directory
ls: cannot access '+%F': No such file or directory

[root@centos84 ~]# ll $(echo $(date +%F).txt)
-rw-r--r-- 1 root root 0 Sep 18 15:05 2021-09-18.txt

[root@centos84 ~]# ll `echo $(date +%F).txt`
-rw-r--r-- 1 root root 0 Sep 18 15:05 2021-09-18.txt

[root@centos84 ~]# ll $(echo `date +%F`.txt)
-rw-r--r-- 1 root root 0 Sep 18 15:05 2021-09-18.txt
```

##### 1.3.5.2 **括号扩展：{}**

{}可以实现打印重复字符串的简化形式

```bash
{元素1,元素2,元素3}
{元素1..元素2}
```

范例：

```bash
echo file{1,3,5} 结果为：file1 file3 file5
rm -f file{1,3,5}
echo {1..10}
echo {a..z}
echo {1..10..2}
echo {a..z}
echo {1..10..2}
echo {000..20..2}
```

范例：关闭和启用{}的扩展功能

```bash
[root@centos84 ~]# echo $-
himBHs
[root@centos84 ~]# echo {1..10}
1 2 3 4 5 6 7 8 9 10
[root@centos84 ~]# set +B
[root@centos84 ~]# echo $-
himHs
[root@centos84 ~]# echo {1..10}
{1..10}
[root@centos84 ~]# set -B
[root@centos84 ~]# echo $-
himBHs
[root@centos84 ~]# echo {1..10}
1 2 3 4 5 6 7 8 9 10
```

### 1.4 tab键补全

内部命令：
外部命令：bash根据PATH环境变量定义的路径，自左而右在每个路径搜寻以给定命令命名的文件，第一次找到的命令即为要执行的命令
命令的子命令补全，需要安装bash-completion
注意：用户给定的字符串只有一唯一对应的命令，直接补全，否则，再次Tab会给出列表

#### 1.4.1 命令行历史

当执行命令后，系统默认会在内存记录执行的命令
当用户正常退出时，会将内存的命令历史存放对应历史文件中，默认是~/.bash_history
登录shell时，会读取命令历史文件中记录下的命令加载到内存中
登录进shell后新执行的命令只会记录在内存的缓存区中；这些命令会正常退出时“追加”至命令历史文件中
利用历史命令。可以用它来重复执行命令，提高输入效率

命令：history

```bash
history [-c] [-d offset] [n]
history -anrw [filename]
history -ps arg [arg...]
```

常用选项：

```bash
-c: 清空命令历史
-d offset: 删除历史中指定的第offset个命令
-a: 追加本次会话新执行的命令历史列表至历史文件
-r: 读历史文件附加到历史列表
-w: 保存历史列表到指定的历史文件
-n: 读历史文件中未读过的行到历史列表
-p: 展开历史参数成多行，但不存在历史列表中
-S: 展开历史参数成一行，附加在历史列表后
```

命令历史相关环境变量

```bash
HISTSIZE：命令历史记录的条数
HISTFILE：指定历史文件，默认为~/.bash_history
HISTFILESIZE：命令历史文件记录历史的条数
HISTTIMEFORMAT=“%F %T `whoami`” 显示时间和用户
HISTIGNORE=“str1:str2*:…” 忽略str1命令，str2开头的历史
HISTCONTROL：控制命令历史的记录方式
```

ignoredups 是默认值，可忽略重复的命令，连续且相同为“重复”
ignorespace 忽略所有以空白开头的命令
ignoreboth 相当于ignoredups, ignorespace的组合
erasedups 删除重复命令

**持久保存变量**

以上变量可以 export 变量名=“值” 形式存放在 /etc/profile 或 ~/.bash_profile
范例：

```bash
[root@centos84 ~]# cat .bash_profile
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
	. ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/bin

export PATH
export HISTCONTROL=ignoreboth 
export HISTTIMEFORMAT="%F %T "

4.18.0-305.3.1.el8.x86_64
[root@centos84 ~]# history 
    1  2021-12-02 18:14:21 ls /etc/
    2  2021-12-02 18:14:25 date
    3  2021-12-02 18:14:49 uname -a
    4  2021-12-02 18:14:51 uname -r
    5  2021-12-02 18:15:04 history 
```

#### 1.4.2 调用命令历史

```bash
#重复前一个命令方法 
重复前一个命令使用上方向键，并回车执行 按 
!! 并回车执行 输入 
!-1 并回车执行 按 Ctrl+p 并回车执行 
!:0 执行前一条命令（去除参数） 
!n 执行history命令输出对应序号n的命令 
!-n 执行history历史中倒数第n个命令 
!string 重复前一个以“string”开头的命令

!?string 重复前一个包含string的命令 
!string:p 仅打印命令历史，而不执行 
!$:p 打印输出 !$ （上一条命令的最后一个参数）的内容 
!*:p 打印输出 
!*（上一条命令的所有参数）的内容 
^string 删除上一条命令中的第一个string 
^string1^string2 将上一条命令中的第一个string1替换为string2 !:gs/string1/string2 将上一条命令中所有的string1都替换为 string2 
使用up（向上）和down（向下）键来上下浏览从前输入的命令 
ctrl-r来在命令历史中搜索命令 （reverse-i-search）`’： 
Ctrl+g：从历史搜索模式退出

#要重新调用前一个命令中最后一个参数 
!$ 表示 Esc, . 点击Esc键后松开，然后点击 . 键 
Alt+ . 按住Alt键的同时点击 . 键 
注意：Alt组合快捷键经常和其它软件冲突
xshell中启动 alt 键

command !^ 利用上一个命令的第一个参数做command的参数 
command !$ 利用上一个命令的最后一个参数做command的参数 
command !* 利用上一个命令的全部参数做command的参数 
command !:n 利用上一个命令的第n个参数做command的参数 
command !n:^ 调用第n条命令的第一个参数 
command !n:$ 调用第n条命令的最后一个参数 
command !n:m 调用第n条命令的第m个参数 
command !n:* 调用第n条命令的所有参数 
command !string:^ 从命令历史中搜索以 string 开头的命令，并获取它的第一个参数 
command !string:$ 从命令历史中搜索以 string 开头的命令,并获取它的最后一个参数 
command !string:n 从命令历史中搜索以 string 开头的命令，并获取它的第n个参数 
command !string:* 从命令历史中搜索以 string 开头的命令，并获取它的所有参数
```

## 2.获得帮助

多层次的帮助

- whatis
- command --help
- man and info
- /usr/share/doc/
- Red Hat documentation 、Ubuntu documentation
- 软件项目网站
- 其它网站
- 搜索

### 2.1 whatis

whatis 使用数据库来显示命令的简短描述

刚安装后不可立即使用，需要制作数据库

```bash
#CentOS 7 版本以后 
mandb 
#CentOS 6 版本之前 
makewhatis
```

```bash
[root@centos84 ~]# whatis cal
cal (1)              - display a calendar
cal (1p)             - print a calendar
[root@centos84 ~]# man -f cal
cal (1)              - display a calendar
cal (1p)             - print a calendar
```

```bash
[root@centos8 ~]#whatis ls 
ls: nothing appropriate. 
#生成man相关数据库 
[root@centos8 ~]#mandb 
Processing manual pages under /usr/share/man... 
Updating index cache for path `/usr/share/man/mann'. Wait...done. 
Checking for stray cats under /usr/share/man... 
...省略... 
0 old database entries were purged. 
[root@centos8 ~]#whatis ls 
ls (1) - list directory contents
```

### 2.2 查看命令的帮助

#### 2.2.1 内部命令：

- help COMMAND
- man bash

```bash
[root@centos84 ~]# type history    先判断是否内部命令
history is a shell builtin
[root@centos84 ~]# help history 
```

#### 2.2.2 外部命令和软件：

- COMMAND --help 或 COMMAND -h
- 使用手册(manual)

man COMMAND

- 信息页

info COMMAND

- 程序自身的帮助文档

README
INSTALL
ChangeLog

- 程序官方文档
- 官方站点：Documentation
- 发行版的官方文档
- (7) Google

### 2.3 --help 或 -h 选项

显示用法总结和参数列表，大多数命令使用，但并非所有的
范例：

```bash
[root@centos84 ~]# date --help
Usage: date [OPTION]... [+FORMAT]
  or:  date [-u|--utc|--universal] [MMDDhhmm[[CC]YY][.ss]]
Display the current time in the given FORMAT, or set the system date.

[root@centos84 ~]# cal -h

Usage:
 cal [options] [[[day] month] year]
 cal [options] <timestamp|monthname>

[root@centos84 ~]# openssl --help
Invalid command '--help'; type "help" for a list.

[root@centos84 ~]# date -h
date: invalid option -- 'h'
Try 'date --help' for more information.

[root@centos84 ~]# shutdown -h
Shutdown scheduled for Fri 2021-12-03 12:46:08 CST, use 'shutdown -c' to cancel.
```

1、显示当前时间，格式：2016-06-18 10:20:30

```bash
[root@centos84 ~]# date +"%Y-%m-%d %T" 
2021-12-03 12:51:46
```

2、显示前天是星期几

```bash
[root@centos84 ~]# date -d -2day +%u
3
```

3、设置当前日期为2019-08-07 06:05:10

```bash
[root@centos84 ~]# date +"%Y-%m-%d %H:%M:%S"
2021-12-03 13:05:46
```

### 2.4 man命令

man 提供命令帮助的文件,手册页存放在/usr/share/man
几乎每个命令都有man的“页面”
中文man需安装包

- man-pages
- man-pages-zh-CN

man 页面分组为不同的“章节”,统称为Linux手册，man 1 man
1：用户命令
2：系统调用
3：C库调用
4：设备文件及特殊文件
5：配置文件格式
6：游戏
7：杂项
8：管理类的命令
9：Linux 内核API
man命令的配置文件：

```bash
#CentOS 6 之前版 man 的配置文件 
/etc/man.config 
#CentOS 7 之后版 man 的配置文件 
/etc/man_db.conf 
#ubuntu man 的配置文件 
/etc/manpath.config
```

格式：

```bash
MANPATH /PATH/TO/SOMEWHERE #指明man文件搜索位置
```

也可以指定位置下搜索COMMAND命令的手册页并显示

```bash
man -M /PATH/TO/SOMEWHERE COMMAND
```

查看man手册页

```bash
man [章节] keyword
```

man 帮助段落说明

- NAME 名称及简要说明
- SYNOPSIS 用法格式说明
- [] 可选内容
- <> 必选内容
- a|b 二选一
- { } 分组
- … 同一内容可出现多次
- DESCRIPTION 详细说明
- OPTIONS 选项说明
- EXAMPLES 示例
- FILES 相关文件
- AUTHOR 作者
- COPYRIGHT 版本信息
- REPORTING BUGS bug信息
- SEE ALSO 其它帮助参考

列出所有帮助

```bash
man -a keyword
```

搜索man手册

```bash
#列出所有匹配的页面，使用 whatis 数据库 
man -k keyword
```

相当于 whatis

```bash
man -f keyword
```

打印man帮助文件的路径

```bash
man -w [章节] keyword
```

范例：

```bash
[root@centos84 ~]# man -w 1 passwd
/usr/share/man/man1/passwd.1.gz
[root@centos84 ~]# whatis passwd
passwd (5)           - password file
openssl-passwd (1ssl) - compute password hashes
passwd (1)           - update user's authentication tokens
[root@centos84 ~]# man 1ssl openssl-passwd
[root@centos84 ~]# dnf install man-pages
[root@centos84 ~]# man 7 ascii
[root@centos84 ~]# man 7 utf8
```

man命令的操作方法：使用less命令实现

- space, ^v, ^f, ^F: 向文件尾翻屏

- b, ^b: 向文件首部翻屏

- d, ^d: 向文件尾部翻半屏

- u, ^u: 向文件首部翻半屏

- RETURN, ^N, e, ^E or j or ^J: 向文件尾部翻一行

- y or ^Y or ^P or k or ^K：向文件首部翻一行

- q: 退出

- #：跳转至第#行

- 1G: 回到文件首部

- G：翻至文件尾部

- /KEYWORD

  以KEYWORD指定的字符串为关键字，从当前位置向文件尾部搜索；不区分字符大小写

  n：下一个

  N：上一个

- ?KEYWORD

​		以KEYWORD指定的字符串为关键字，从当前位置向文件首部搜索；不区分字符		大小写
​		n：跟搜索命令同方向，下一个
​		N：跟搜索命令反方向，上一个
​		范例：

1、在本机字符终端登录时，除显示原有信息外，再显示当前登录终端号，主机		名和当前时间

```bash
[root@centos84 ~]# cat /etc/issue
\S
Kernel \r on an \m
terminal: \l
hostname: \n
current time: \d \t
```

2、今天18：30自动关机，并提示用户

```bash
shutdown  -P(power off)  "18:30"
```

### 2.5 info

man常用于命令参考 ，GNU工具 info 适合通用文档参考
没有参数,列出所有的页面
info 页面的结构就像一个网站
每一页分为“节点”
链接节点之前 *
info 命令格式

```bash
info [ 命令 ]
```

- 方向键，PgUp，PgDn 导航
- Tab键 移动到下一个链接
- d 显示主题目录
- Home 显示主题首部
- Enter进入 选定链接
- n/p/u/l 进入下/前/上一层/最后一个链接
- s 文字 文本搜索
- q 退出 info

### 2.6 Linux 安装提供的本地文档获取帮助

Applications -> documentation->help（centos7）
System->help（centos6）

### 2.7 命令自身提供的官方使用指南

/usr/share/doc 目录
多数安装了的软件包的子目录,包括了这些软件的相关原理说明
常见文档：README INSTALL CHANGES
不适合其它地方的文档的位置
配置文件范例
HTML/PDF/PS 格式的文档
授权书详情

### 2.8 系统及第三方应用官方文档

#### 2.8.1 通过在线文档获取帮助

 http://www.github.com
 https://www.kernel.org/doc/html/latest/
 http://httpd.apache.org
 http://www.nginx.org
 https://mariadb.com/kb/en
 https://dev.mysql.com/doc/
 http://tomcat.apache.org
 https://jenkins.io/zh/doc/
 https://kubernetes.io/docs/home/
 https://docs.openstack.org/train/
 http://www.python.org
 http://php.net

#### 2.8.2 Linux官方在线文档和知识库

通过发行版官方的文档光盘或网站可以获得安装指南、部署指南、虚拟化指南等
 http://kbase.redhat.com
 http://www.redhat.com/docs
 http://access.redhat.com
 https://help.ubuntu.com/lts/serverguide/index.html
 http://tldp.org

#### 2.8.3 红帽全球技术支持服务

rhn.redhat.com或者本地卫星服务器/代理服务器
RHN账户为及其注册和基于网络管理的RHN用户
sosreport 收集所有系统上的日志信息的工具，并自动打成压缩包，方便技术支持人员和红帽全球支持
提供分析问题依据
范例：

```bash
[root@centos84 ~]# dnf -y install sos
[root@centos84 ~]# sosreport 
Please note the 'sosreport' command has been deprecated in favor of the new 'sos' command, E.G. 'sos report'.
Redirecting to 'sos report '

sosreport (version 4.1)

This command will collect diagnostic and configuration information from
this CentOS system and installed applications.

An archive containing the collected information will be generated in
/var/tmp/sos.85b3xk5e and may be provided to a CentOS support
representative.

Any information provided to CentOS will be treated in accordance with
the published support policies at:

        Community Website : https://www.centos.org/

The generated archive may contain data considered sensitive and its
content should be reviewed by the originating organization before being
passed to any third party.

No changes will be made to system configuration.

Press ENTER to continue, or CTRL-C to quit.

Please enter the case id that you are generating this report for []: 2

 Setting up archive ...
 Setting up plugins ...
[plugin:networking] skipped command 'ip -s macsec show': required kmods missing: macsec.  Use '--allow-system-changes' to enable collection.
[plugin:networking] skipped command 'ss -peaonmi': required kmods missing: unix_diag, inet_diag, tcp_diag, udp_diag, af_packet_diag, netlink_diag.  Use '--allow-system-changes' to enable collection.
[plugin:systemd] skipped command 'resolvectl status': required services missing: systemd-resolved. 
[plugin:systemd] skipped command 'resolvectl statistics': required services missing: systemd-resolved. 
 Running plugins. Please wait ...

  Finishing plugins              [Running: host]                                          
  Finished running plugins                                                               
Creating compressed archive...

Your sosreport has been generated and saved in:
	/var/tmp/sosreport-centos84-2-2021-12-03-haofzca.tar.xz

 Size	11.95MiB
 Owner	root
 sha256	a40ac9fef989d5215309ebdab7f9f82bd18a95c44cf8a57350c0ae6fbbfa585f

Please send this file to your support representative.

[root@centos84 ~]# ll /var/tmp/sosreport-centos84-2-2021-12-03-haofzca.tar.xz
-rw------- 1 root root 12525976 Dec  3 14:24 /var/tmp/sosreport-centos84-2-2021-12-03-haofzca.tar.xz
```

### 2.9 网站和搜索

 http://www.google.com

```bash
Openstack filetype:pdf 
rhca site:redhat.com/docs
```


 http://www.slideshare.net

# 2.文件管理

### 1.文件系统目录结构

#### 1.1 系统目录功能

```bash
/etc    保存了linux各种配置文件
/etc/resolv.conf   dns服务器解析域名的配置文件
/etc/hosts   本地解析域名的配置文件
/etc/sysconfig/network-script/ifcfg-网卡名称   配置网卡
/sbin   超级用户才能使用的命令 （根下的是一个软链接文件，链接到/usr/sbin）
/boot   linux开机时加载的一些文件，包括linux内核的一些文件
/bin   普通用户使用的命令（根下的是一个软链接文件，链接到/usr/bin）
/usr   保存了大量的Linux系统文件，类似与Windows系统中c盘中的windows文件
/usr/bin /usr/sbin
/usr/local   程序的安装目录
/home   普通用户的家目录
里面包含了一些环境程序
/root   超级用户的家目录
/dev   存放一些设备文件
/dev/null   linux的黑洞文件，只出不进
/dev/zero   只出不进数据
/dev/random   制造随机数的文件
/var   存放一些可变化的文件，例如日志文件等
/var/tmp   临时存放程序的缓存文件
/lib   是一个库文件，很多命令依赖于库文件（根下的lib也是一个软链接文件，链接到/usr/lib）
/lib64   64位库文件，（同上，根下lib64也是一个软链接文件，链接到/usr/lib64）
/mnt  /media  这两个文件都是一个磁盘的挂载点
/porc   反映了程序运行的状态
/opt   第三方文件的下载目录
/tmp 临时存放文件的地方
/run   程序运行的pid 和一些相关的lock文件
```

#### 1.2 文件系统目录结构

- 以. 开头的文件位隐藏问价
- 蓝色--目录     绿色--可执行文件   红色--压缩文件 浅蓝色--链接文件 灰色--其他文件

#### 1.3  文件类型

​	ls  -l

-  - 普通文件
-  d  目录文件
-  b  块设备(block)
-  c  字符文件(character)
-  l  符号链接文件
-  p  管道文件pipe
-  s  套接字文件socket

#### 1.4 Linux中常用的一些简单文件管理

```bash
cd    目录切换命令
cd ..    表示
cd . 表示再当前目录
cd - 表示返回上一次所在的路径
pwd    显示当前所在路径
mkdir    文件或目录的创建
mkdir    后面加参数，参数即目标目录
mkdir -p    为递归创建目录，例如/a/b/c
再根下a目录下创建b目录，然后再b目录下创建c目录，如果没有b目录，只能-p，将b和c一块创建
mkdir    {1..100}    在当前目录下创建名字依次为1至100的100个文件夹
touch    创建一个空文件
mv    文件或目录的移动
mv    + 需要移动的文件或目录 + 移动到的目标文件目录
mv    同一个文件夹里移动就是重命名的功能
例如    将 当前目录下的test文件移动到/usr/local/ttt文件夹中
如果local目录下没有ttt文件夹，那么test文件就会被移动到local目录下，并且重命名为ttt
rm    文件或目录的删除
-f    强制删除
-r    递归删除目录及其内容
rm    {1..100}    将1到100 这100 个目录删除
rm    file*     删除已file开头的文件或目录
```

### 2. 文件操作命令

#### 2.1 pwd

查看用户当前所在目录

- -L 显示链接路径（默认）
- -P 显示真实物理路径

```bash
[root@centos8 bin]# ll /bin
lrwxrwxrwx. 1 root root 7 May 11  2019 /bin -> usr/bin
[root@centos8 bin]# pwd -L
/bin
[root@centos8 bin]# pwd -P
/usr/bin
```

#### 2.2 基名字和目录名

基名：basename,路径只取文件名

目录名：dirname,路径只取路径

```bash
[root@centos8 bin]# basename /etc/sysconfig/network
network
[root@centos8 bin]# dirname !*
dirname /etc/sysconfig/network
/etc/sysconfig
```

#### 2.3 cd

cd: cd [-L|[-P [-e]] [-@]] [dir]
Change the shell working directory.

**快捷用法**

- 切换至父目录 cd ..
- 切换至当前工作目录 cd .
- 切换至上一次工作目录 cd -

注意：这里上一次工作目录是保存在$OLDPWD变量当中的，所以只能切换到上一次工作目录，而无法返回再往前的目录

```bash
[root@centos8 bin]# cd /
[root@centos8 /]# echo $OLDPWD
/bin
[root@centos8 /]# cd -
/bin
```

#### 2.4 ls

ls ：列出当前目录内容或指定目录内容

```bash
SYNOPSIS
ls [OPTION]... [FILE]...
```

**常用选项**

- -a 列出所有文件，包括隐藏文件
- -l 显示更为详细的信息
- -R 递归展示目录
- -d 展示目录而不显示目录的内容
- -s 按照大小排序
- -t 按照mtime排序

说明：

ls 查看不同后缀文件时的颜色由 /etc/DIR_COLORS 和@LS_COLORS变量定义

ls是'ls --color=auto'别名

```bash
[root@centos8 etc]# ls -ld /etc/
drwxr-xr-x. 78 root root 8192 Dec  6 06:19 /etc/
```

#### 2.4 stat

stat：显示文件或文件系统的状态

文件三个时间戳及其含义：

- atime 访问时间，如执行读取文件操作可以改变访问时间
- mtime 修改时间 ，如利用vim修改文件时时间会发生改变
- ctime 改变时间，元数据（文件属性）发生改变时

```bash
[root@centos8 ~]# stat ls
  File: ls
  Size: 116       	Blocks: 8          IO Block: 4096   regular file
Device: 802h/2050d	Inode: 201327540   Links: 1
Access: (0600/-rw-------)  Uid: (    0/    root)   Gid: (    0/    root)
Context: unconfined_u:object_r:admin_home_t:s0
Access: 2020-11-29 22:04:36.787356892 +0800
Modify: 2020-11-29 22:04:36.787356892 +0800
Change: 2020-11-29 22:04:36.787356892 +0800
 Birth: -
 
 #使用ll就能查看到文件的元数据
 [root@centos8 ~]# ll
-rw-------. 1 		root root 		1548 Nov 21 03:24  anaconda-ks.cfg
-rw-r--r--. 1 		root root    		0 Dec  7 05:01 lx.txt
#权限	引用计数	  所有者和所属组	 	大小	  修改时间	   文件名
```

#### 2.5 file

**file**：**判断文件类型**

```bash
file [options] <filename>...
```

**常用选项**：

- -b 只显示判断的文件类型，不显示文件名
- -f 判断文件File中文件名的类型
- -F 使用指定分隔符号替换输出文件名后默认的”:”分隔符
- -L 查看对应软链接对应文件的文件类型

```bash
[root@centos8 ~]# cat lx.txt 
/root/lx.txt
/root/ls
/etc/sysconfig
[root@centos8 ~]# file -f lx.txt 
/root/lx.txt:   ASCII text
/root/ls:       ASCII text
/etc/sysconfig: directory
[root@centos8 ~]# file -L /bin/
/bin/: directory
[root@centos8 ~]# file /bin
/bin: symbolic link to usr/bin
```

注意：windows文件系统和Linux是不同的，即使windows文本格式文件在linux系统中以16进制查看所显示内容也是有区别的，所以尽量不要在Linux系统中使用windows的文件，如要使用也最好进行文件转换

```bash
[root@centos8 data]#cat linux.txt
a
b
c
[root@centos8 data]#cat win.txt
a
b
c[root@centos8 data]#file win.txt linux.txt
win.txt:   ASCII text, with CRLF line terminators
linux.txt: ASCII text
[root@centos8 data]#hexdump -C linux.txt
00000000  61 0a 62 0a 63 0a                                 |a.b.c.|
00000006
[root@centos8 data]#hexdump -C win.txt
00000000  61 0d 0a 62 0d 0a 63                             |a..b..c|
00000007
#安装转换工具
[root@centos8 data]#dnf -y install dos2unix
#将Windows的文本格式转换成的Linux文本格式
[root@centos8 data]#dos2unix win.txt
dos2unix: converting file win.txt to Unix format...
[root@centos8 data]#file win.txt
win.txt: ASCII text
#将Linux的文本格式转换成Windows的文本格式
[root@centos8 data]#unix2dos win.txt
unix2dos: converting file win.txt to DOS format...
[root@centos8 data]#file win.txt
win.txt: ASCII text, with CRLF line terminators
```

**转换文件字符集编码**

```bash
#显示支持字符集编码列表
[root@centos8 ~]#iconv -l
#windows10上文本默认的编码ANSI（GB2312）
[root@centos8 data]#file windows.txt
windows.txt: ISO-8859 text, with no line terminators
[root@centos8 data]#echo $LANG
en_US.UTF-8
#默认在linux无法正常显示文本内容
[root@centos8 data]#cat windows11.txt
▒▒▒▒▒▒[root@centos8 data]#
#将windows10上文本默认的编码ANSI（GB2312）转换成UTF-8
[root@centos8 data]#iconv -f gb2312 windows.txt -o windows1.txt
[root@centos8 data]#cat windows1.txt
123[root@centos8 data]#ll windows1.txt
-rw-r--r-- 1 root root 12 Mar 23 10:13 windows1.txt
[root@centos8 data]#file windows1.txt
windows1.txt: UTF-8 Unicode text, with no line terminators
#将UTF-8转换成windows10上文本默认的编码ANSI（GB2312）
[root@centos8 data]#iconv -f utf8 -t gb2312 windows1.txt -o windows11.txt
[root@centos8 data]#file windows11.txt
windows11.txt: ISO-8859 text, with no line terminators
```

示例

```bash
#将windows10上文本默认的编码ANSI（GB2312）转换成UTF-8
[15:34:50 root@centos8 ~]#iconv -f gb2312 win.txt -o win2.txt  
[15:34:50 root@centos8 ~]#file linux.txt
linux.txt: ASCII text
[15:34:31 root@centos8 ~]#file windows.txt
windows.txt: ASCII text, with CRLF line terminators
#将windows的文本格式转换成Linux的文本格式
[15:35:26 root@centos8 ~]#dos2unix windows.txt
dos2unix: converting file windows.txt to Unix format...
[15:36:00 root@centos8 ~]#file windows.txt
windows.txt: ASCII text
```

示例

```bash
[root@centos8 ~]#cat list.txt
/etc/
/bin
/etc/issue
[root@centos8 ~]#file -f list.txt
/etc/:     directory
/bin:       symbolic link to usr/bin
/etc/issue: ASCII text
```

#### 2.7 通配符

用来匹配符合条件的文件

**常见的通配符**

```bash
* 匹配零个或多个字符，但不匹配 "." 开头的文件，即隐藏文件
? 匹配任何单个字符
~ 当前用户家目录
~user 用户user家目录
~+和.   当前工作目录
~-   前一个工作目录
[0-9] 匹配数字范围
[a-z] 匹配小写字母范围
[A-Z] 匹配大写字母范围
[number] 匹配列表中的任何的一个字符
[^number] 匹配列表中的所有字符以外的字符
```

系统中预定义的字符类：man 7 glob

```bash
[:digit:]：任意数字，相当于0-9
[:lower:]：任意小写字母,表示 a-z
[:upper:]: 任意大写字母,表示 A-Z
[:alpha:]: 任意大小写字母
[:alnum:]：任意数字或字母
[:blank:]：水平空白字符
[:space:]：水平或垂直空白字符
[:punct:]：标点符号
[:print:]：可打印字符
[:cntrl:]：控制（非打印）字符
[:graph:]：图形字符
[:xdigit:]：十六进制字符
```

#### 2.8 touch

touch：创建空文件和改变文件时间

```bash
 touch [OPTION]... FILE...
```

选项：

- -a 仅改变 atime和ctime
- -m 仅改变 mtime和ctime
- -t [[CC]YY]MMDDhhmm[.ss] 指定atime和mtime的时间戳
- -c 如果文件不存在，则不予创建

```bash
#touch命令最常用的用法就是用来创建空文件
[root@centos8 data]#touch `date -d "-1 day" +%F_%T`.log
[root@centos8 data]#ll
-rw-r--r-- 1 root root   0 Dec  6 22:00 2020-12-05_22:00:15.log
[root@centos8 data]#touch $(date -d "1 year" +%F_%T).log
[root@centos8 data]#ll
-rw-r--r-- 1 root root   0 Dec  6 22:00 2021-12-06_22:00:38.log
```

#### 2.9 cp

cp：复制文件或目录

```bash
cp [OPTION]... [-T] SOURCE DEST
cp [OPTION]... SOURCE... DIRECTORY
cp [OPTION]... -t DIRECTORY SOURCE...
alias cp='cp -i'
```

常用选项

- -i 如果目标已存在，覆盖前提示是否覆盖  （如果不想提示，使用\cp 原命令）

- -a 归档，相当于-dR --preserv=all，常用于备份功能
- -i 如果目标已存在，覆盖前提示是否覆盖
- -r, -R 递归复制目录及内部的所有内容
- -b 目标存在，覆盖前先备份，默认形式为 filename~ ,只保留最近的一个备份
- --backup=numbered 目标存在，覆盖前先备份加数字后缀，形式为 filename.# ，可以保留多 个版本
- -v 显示复制过程

```bash
#复制文件
[root@centos8 /]# cp -a /root/lx.txt /data/
[root@centos8 /]# ls /data/
lx.txt
#复制目录
[root@centos8 /]# cp -r /etc/sysconfig/ /data
[root@centos8 /]# ll /data/
total 8
-rw-r--r-- 1 root root  253 Dec  6 21:55 lx.txt
drwxr-xr-x 6 root root 4096 Dec  6 22:15 sysconfig
```

#### 2.10 mv

mv：对文件或目录进行移动和改名

```bash
mv [OPTION]... [-T] SOURCE DEST
mv [OPTION]... SOURCE... DIRECTORY
mv [OPTION]... -t DIRECTORY SOURCE...
```

常用选项

- -i 交互式
- -f 强制
- -b 目标存在，覆盖前先备份

使用rename可以完成对文件的批量改名

```bash
SYNOPSIS
rename [options] expression replacement file...

[root@centos8 data]# ls
file10.txt  file2.txt  file4.txt  file6.txt  file8.txt  lx.txt
file1.txt   file3.txt  file5.txt  file7.txt  file9.txt  sysconfig
[root@centos8 data]# rename '.txt' '' *.txt
[root@centos8 data]# ls
file1  file10  file2  file3  file4  file5  file6  file7  file8  file9  lx  sysconfig
[root@centos8 data]# 
```

#### 2.11 rm

rm：删除文件或目录

```bash
rm [OPTION]... [FILE]...
```

常用选项

- -i 交互式
- -f 强制删除
- -r 递归

删除特殊文件：-f

```bash
1.rm -- -foo
2.rm ./-foo
[root@centos8 data]# touch ./-f
[root@centos8 data]# ls
-f
[root@centos8 data]# rm -rf -- -f
```

使用shred可以对磁盘惊醒覆盖，从而彻底删除文件

```bash
shred  [OPTION]... FILE...
OPTION
-z 最后一次覆盖添加0，以隐藏覆盖操作
-v 显示操作进度
-u 覆盖后阶段并删除文件
-n# 指定覆盖文件内容次数（默认3次）
```

#### 2.14 目录命令

##### 2.12.1 tree：显示目录树

常用选项

- -d: 只显示目录
- -L level：指定显示的层级数目
- -P pattern: 只显示由指定wild-card pattern匹配到的路径

##### 2.12.2 mkdir

mkdir：创建目录

常用选项：

- -v: 显示详细信息

##### 2.12.3 rmdir

rmdir：删除空目录，使用较少

### 3.文件元数据和节点表结构

#### 3.1文件系统组成

##### 3.1.1 **inode结构**

inode块中保存的是文件的属性信息，包括以下内容

- inode number 节点号
- 文件类型
- 权限
- UID
- GID
- 链接数
- 文件大小和atime、ctime、mtime
- 文件内容的真正位置
- 其他文件相关数据

##### 3.1.2 **目录** 

目录是个特殊文件，檔案系統会分配一个 inode 与至少一个 block 给目录。其中，inode 记录目录相关权限与属性，和可分配到的block块的号码； 而 block 则是记录在这个目录下的文件名占用的inode number节点号。

**cp和inode**

- 分配一个空闲的inode号，在inode表中生成新条目
- 在目录中创建一个目录项，将名称与inode编号关联
- 拷贝数据生成新的文件

**rm和inode**

- 链接数递减，从而释放的inode号可以被重用
- 把数据块放在空闲列表中
- 删除目录项
- 数据实际上不会马上被删除，但当另一个文件使用数据块时将被覆盖

**mv和inode**

- 如果mv命令的目标和源在相同的文件系统，作为mv 命令

  用新的文件名创建对应新的目录项

  删除旧目录条目对应的旧的文件名

  不影响inode表（除时间戳）或磁盘上的数据位置：没有数据 被移动！

- 如果目标和源在一个不同的文件系统， mv相当于cp和rm

#### 3.2 硬链接(Hard Link)

硬链接本质上就给一个文件起一个新的名称，实质是同一个文件

硬链接特性

- 创建硬链接会在对应的目录中增加额外的记录项以引用文件
- 对应于同一文件系统上一个物理文件
- 每个目录引用相同的inode号
- 创建时链接数递增
- 删除文件时：rm命令递减计数的链接，文件要存在，至少有一个链接数，当链接数为零时，该文 件被删除
- 不能跨越驱动器或分区
- 不支持对目录创建硬链接

链接目录时会多出一个.，而.又会让目录产生一个新的链接，这会导致错误循环问题

```bash
ln [OPTION]... TARGET... DIRECTORY
```

#### 3.3 符号 symbolic （或软 soft Link）链接

一个符号链接指向另一个文件,就像 windows 中快捷方式，软链接文件和原文件本质上不是同一个文件

```bash
ln -s [OPTION]... TARGET... DIRECTORY
```

软链接特点

- 可以对目录创建软链接
- 可以跨分区的文件实现
- 软链接如果使用相对路径，是相对于原文件的路径，而非相对于当前目录
- 指向的是另一个文件的路径；其大小为指向的路径字符串的长度；不增加或减少目标文件inode的 引用计数
- 一个符号链接的内容是它引用文件的名称
- 删除软链接的时候，加上 /  会把该目录里的文件也删 

#### 3.4 软链接和硬链接的区别

1. 本质：

   硬链接：本质是同一个文件

   软链接：本质不是同一个文件

2. 跨区

   硬链接：不支持

   软链接：支持

3. inode

硬链接：相同

软链接：不同

1. 链接数

   硬链接：创建新的硬链接,链接数会增加,删除硬链接,链接数减少

   软链接：创建或删除,链接数不会变化

2. 目录

   硬链接：不支持

   软链接：支持

3. 相对路径

   硬链接：原始文件相对路径是相对于当前工作目录

   软链接：原始文件的相对路径是相对于链接文件的相对路径

4. 删除源文件

   硬链接：只是链接数减一,但链接文件的访问不受影响

   软链接：链接文件将无法访问

5. 文件类型

   硬链接：和源文件相同

   软链接：链接文件,和源文件无关

# 3.IO重定向&权限

## 1.标准输入和输出

- 标准输入（STDIN）-- 文件描述符：0 默认接受来自终端窗口
- 标准输出（STDOUT）-- 文件描述符：1 默认输出到终端窗口
- 标准错误（STDERR）-- 文件描述符：2 默认输出到终端窗口

```bash
使用ps aux无法看到有些程序路径，这时如何查找到其路径？
#利用ps aux找到其PID，使用echo $$命令输出当前终端PID

[root@Centos7 ~]# echo $$
1708
#proc目录下exe指向的就是程序所在路径
[root@Centos7 ~]# ll /proc/$$ | grep exe
lrwxrwxrwx. 1 root root 0 Dec 14 06:12 exe -> /usr/bin/bash
#fd（file descriptor）文件夹中可以查看当前程序的文件描述符
[root@Centos7 ~]# cd /proc/$$/fd
[root@Centos7 fd]# ll
total 0
lrwx------. 1 root root 64 Dec 14 00:44 0 -> /dev/pts/0
lrwx------. 1 root root 64 Dec 14 00:44 1 -> /dev/pts/0
lrwx------. 1 root root 64 Dec 14 00:44 2 -> /dev/pts/0
lrwx------. 1 root root 64 Dec 14 06:24 255 -> /dev/pts/0


文件描述符查看(默认当前终端窗口)
[root@Centos7 fd]# ll /dev/std*
lrwxrwxrwx. 1 root root 15 Dec 13 22:34 /dev/stderr -> /proc/self/fd/2
lrwxrwxrwx. 1 root root 15 Dec 13 22:34 /dev/stdin -> /proc/self/fd/0
lrwxrwxrwx. 1 root root 15 Dec 13 22:34 /dev/stdout -> /proc/self/fd/1

[root@Centos7 fd]# ll /proc/self/fd
total 0
lrwx------. 1 root root 64 Dec 14 06:57 0 -> /dev/pts/0
lrwx------. 1 root root 64 Dec 14 06:57 1 -> /dev/pts/0
lrwx------. 1 root root 64 Dec 14 06:57 2 -> /dev/pts/0
lr-x------. 1 root root 64 Dec 14 06:57 3 -> /proc/2470/fd
```

## 2. I/O重定向（redirect）

### 2.1 标准输出重定向

**会覆盖文件的操作符**

```bash
1> 或  >	把标准输出（STDOUT）重定向到文件
2>		把标准错误（STDERR）重定向到文件
&>		把所有输出重定向到文件中
```

**追加到文件的操作符**

```bash
>>		追加标准输出（STDOUT）到文件
2>>		追加标准错误（STDERR）到文件
```

**将标准输出和错误输出定向到不同的位置**

```bash
COMMAND > /path/to/file.out 2> /path/to/err.out
```

**将标准输出和错误输出进行合并重定向**

```bash
COMMAND	> /path/to/file.out 2>&1
COMMAND	>> /path/to/file.out 2>&1
#范例
[root@Centos7 data]# ll /data/ xxx > /data/all.log 2>&1
[root@Centos7 data]# ll
total 4
-rw-r--r--. 1 root root 113 Dec 15 22:56 all.log
[root@Centos7 data]# cat all.log 
ls: cannot access xxx: No such file or directory
/data/:
total 4
-rw-r--r--. 1 root root 49 Dec 15 22:56 all.log
```

**合并多个程序**

```bash
（CMD1;CMD2...）or {CMD1;CMD2;...;} > /path/to/file.out
#范例
[root@Centos7 data]# (ls /data/;date) > /data/lx.txt
[root@Centos7 data]# ll
total 8
-rw-r--r--. 1 root root 113 Dec 15 22:56 all.log
-rw-r--r--. 1 root root  44 Dec 15 22:58 lx.txt
[root@Centos7 data]# cat lx.txt 
all.log
lx.txt
Tue Dec 15 22:58:48 CST 2020
```

**/dev/null用法**

```bash
#/dev/null表示的是一个黑洞，类似于windows中回收站
#可以使用/dev/null来清除一些大文件
[root@Centos7 data]# cat /dev/null > /data/lx.txt 
[root@Centos7 data]# cat lx.txt 
[root@Centos7 data]# 
```

**输出重定向到同一文件中的不同写法**

```bash
[root@Centos7 data]# ls /data/ /xxx > /data/all.log 2>&1
[root@Centos7 data]# ls /data/ /xxx 2> /data/all.log 1>&2
[root@Centos7 ~]# ls /data/ /xxx &> /data/all.log
#命令从左往右顺序执行下面命令会先输出标准输出在追加至文件
[root@Centos7 ~]# ls /data/ /xxx 2>&1 > /data/all.log
ls: cannot access /xxx: No such file or directory
```

### 2.2 **标准输入重定向**

#### 2.2.1 **tr命令**

tr ：翻译，压缩和/或删除标准输入中的字符，
输出至标准输出。

```bash
tr [OPTION]... SET1 [SET2]
```

常用选项

- -d, --delete delete characters in SET1, do not translate
- -s, --squeeze-repeats 把重复的字符用单独一个字符表示，去重
- -t, --truncate-set1 削减 SET1 指定范围，使之与 SET2 设定长度相等
- -c, -C, --complement use the complement of SET1（反选，也就是符合 SET1 的部份不做处理）

\NNN character with octal value NNN (1 to 3 octal digits)

\\ backslash

\a audible BEL

\b backspace

\f form feed

\n new line

\r return

\t horizontal tab

\v vertical tab

[:alnum:]：字母和数字

[:alpha:]： 字母

[:digit:]： 数字

[:lower:]：小写字母

[:upper:]：大写字母

[:space:]：空白字符

[:print:]： 可打印字符

[:punct:]：标点符号

[:graph:]：图形字符

[:cntrl:]： 控制（非打印）字符

[:xdigit:]：十六进制字符

范例

```bash
#把/etc/issue中的小写字符都转换成大写字符
[root@Centos7 data]# tr 'a-z' 'A-Z' < issue
[root@Centos7 data]# tr [:lower:] [:upper:] < issue
```



#### **2.2.2 输入重定向**

单行重定向

利用“<” 可以将标准输入重定向

常见的支持输入重定向命令：cat、rm、mail、bc、tr等



```bash
#使用输入重定向计算1-10之和
[root@Centos7 data]# seq -s + 1 10 > cal.txt
[root@Centos7 data]# bc < cal.txt 
55
```

多行重定向

使用 "<<终止词" 命令从键盘把多行重导向给STDIN，直到终止词位置之前的所有文本都发送给 STDIN，有时被称为就地文本（here documents）

其中终止词可以是任何一个或多个符号，比如：!，@，$，EOF（End Of File）等，其中EOF 比较常用

范例：

```bash
#通过发送邮件测试
#安装邮件服务
[root@centos8 ~]# yum -y install postfix
#启动邮件过程报错
[root@centos8 ~]# systemctl start postfix
Job for postfix.service failed because the control process exited with error code.
See "systemctl status postfix.service" and "journalctl -xe" for details.
#按照要求查看journalctl -xe
fatal: file /etc/postfix/main.cf: parameter mydomain: bad parameter value: 2
#打开配置文件查看如下变量引用了mydomain,却没有声明，推测变量未声明使用变量导致报错
mydestination = $myhostname, localhost.$mydomain, localhost
#将mydomain = domain.tld前注释取消，启动邮件服务成功
[root@centos8 ~]# systemctl start postfix
[root@centos8 ~]# ss -ntl
State            Recv-Q           Send-Q                     Local Address:Port                     Peer Address:Port          
LISTEN           0                128                              0.0.0.0:5355                          0.0.0.0:*             
LISTEN           0                128                              0.0.0.0:80                            0.0.0.0:*             
LISTEN           0                128                              0.0.0.0:22                            0.0.0.0:*             
LISTEN           0                100                            127.0.0.1:25                            0.0.0.0:*             

#使用root用户发送一封邮件给git用户
#可以通过PS2修改输入提示符
[root@centos8 ~]# mail -s test git <<EOF
> hello git
> i am  root
> how are you?
> EOF
#通过外部邮件发送
[root@centos8 ~]#yum  install  mailx -y
```



## 3.管道

### 3.1管道符

“|”：使用该符号来连接多个命令

```bash
COMMAND1 | COMMAND2 | COMMAND3
```

说明

- 将COMMAND1的STDOUT发送给COMMAND2的STDIN
- 所有命令会在当前shell进程中的子shell进程中执行
- 组合多种工具功能

STDERR默认不能通过管道转发，可以利用2>&1或着|&实现

范例

```bash
[root@centos8 data]# ls /data/ /err | tr 'a-z' 'A-Z'
ls: cannot access '/err': No such file or directory
/DATA/:
[root@centos8 data]# ls /data/ /err 2>&1 | tr 'a-z' 'A-Z'
LS: CANNOT ACCESS '/ERR': NO SUCH FILE OR DIRECTORY
/DATA/:
```

### 3.2 管道中的 - 符号

管道中有时会使用 - 符号

示例： 将 /home 里面的文件打包，但打包的数据不是记录到文件，而是传送到 stdout，经过管道后，将 tar - cvf - /home 传送给后面的 tar -xvf - , 后面的这个 - 则是取前一个命令的 stdout， 因此，就不需要使用 临时file了

```bash
tar -cvf - /home | tar -xvf -
```

### 3.3 tee命令

tee：重定向到多个目标(read from standard input and write to standard output and files)

```bash
tee [OPTION]... [FILE]...
```

常用选项

- -a 追加
- 用于保存不同阶段的输出，复杂管道的故障排除，同时查看和记录输出

范例

```bash
[root@centos8 ~]#cat   <<EOF | tee /etc/motd
> welcome
> happy new year
> EOF
welcome
happy new year
```

# 4.用户和权限管理

## 1.系统中用户

### 1.1 用户

Linux中每个用户是通过User Id （UID）来唯一标识的

- 管理员root:0
- 系统用户：1-499 （CentOS 6以前）, 1-999 （CentOS 7以后） 对守护进程获取资源进行权限分配
- 登录用户：500+ （CentOS6以前）, 1000+（CentOS7以后） 给用户进行交互式登录使用

### 1.2 用户组

用户组是通过Group ID（GID） 来唯一标识的。

- 管理员root:0
- 系统组：1-499（CentOS 6以前）, 1-999（CentOS7以后）, 对守护进程获取资源进行权限分 配
- 普通组：500+（CentOS 6以前）, 1000+（CentOS7以后）, 给用户使用

### 1.3 用户和组的关系

- 用户的主要组(primary group)：用户必须属于一个且只有一个主组，默认创建用户时会自动创建 和用户名同名的组，做为用户的主要组，由于此组中只有一个用户，又称为私有组

- 用户的附加组(supplementary group)： 一个用户可以属于零个或多个辅助组，附属组


## 2.用户和组的配置文件

### 2.1 用户和组的主要配置文件

- /etc/passwd
- /etc/shadow
- /etc/group
- /etc/gshadow

**su - [用户名] 却换到普通用户**

### 2.2 passwd文件格式

```bash
[root@Centos7 data]# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
```

依次:用户名、密码、UID、GID、用户全名或信息、用户主目录、用户默认使用shell

### 2.3 shadow文件格式

```bash
[root@Centos7 data]# cat /etc/shadow
user:$6$h1C.cePewZU.s9pw$Nxq5eAgSWeAJ5POU55DBI.awbioC4I12Y1zj9IJpZ2EY4SttRQakRxgN.GUz8wH6BYZbtc9AOT98WtOqW1Z43/::0:99999:7:::
```

用户名

用户密码:一般用sha512加密

从1970年1月1日起到密码最近一次被更改的时间

密码再过几天可以被变更（0表示随时可被变更）

密码再过几天必须被变更（99999表示永不过期）

密码过期前几天系统提醒用户（默认为一周）

密码过期几天后帐号会被锁定

从1970年1月1日算起，多少天后帐号失效

**生成随机密码**

范例

```bash
#使用tr命令配合/dev/urandom生成随机密码
[root@Centos7 ~]# tr -dc [:alnum:] < /dev/urandom | head -c 9
HSSb6TIwv
[root@Centos7 ~]# openssl rand -base64 9
VrRy07H0KQK4
```

### 2.4 group文件格式

```bash
[root@Centos7 ~]# cat /etc/group
root:x:0:
bin:x:1:
```

群组名称：就是群组名称

群组密码：通常不需要设定，密码是被记录在 /etc/gshadow

GID：就是群组的 ID

以当前组为附加组的用户列表(分隔符为逗号)

### 2.5 gshdow文件格式

```bash
[root@Centos7 ~]# cat /etc/gshadow
root:::
```

群组名称：就是群的名称

群组密码：

组管理员列表：组管理员的列表，更改组密码和成员

以当前组为附加组的用户列表：多个用户间用逗号分隔

## 3 用户和组管理命令

### 3.1 useradd

useradd：创建或更改用户信息

```bash
useradd [options] LOGIN
```

常用选项

- -r 创建系统用户 CentOS 6之前: ID<500，CentOS 7以后: ID<1000

  -r 不会生成 /home/ 目录里的文件夹

- -s SHELL 指明用户的默认shell程序，可用列表在/etc/shells文件中

- -c "COMMENT“ 用户的注释信息

- -u 指定用户的UID

- -o 配合-u 选项，不检查UID的唯一性

- -d HOME_DIR 以指定的路径(不存在)为家目录

- -D display the current default values(显示useradd默认值)

- -m 创建$HOME目录

范例

```bash
#创建apache系统用户
[root@Centos7 ~]# useradd -r -s /sbin/nologin -d /var/www -c 'apache' apache
```

useradd 命令默认值设定由/etc/default/useradd定义

```bash
[root@Centos7 ~]# cat /etc/default/useradd 
# useradd defaults file
GROUP=100
HOME=/home
INACTIVE=-1					#对应/etc/shadow文件第7列，即用户密码过期的宽限期
EXPIRE=						 #对应/etc/shadow文件第8列，即用户帐号的有效期
SHELL=/bin/bash
SKEL=/etc/skel				#用户家目录的模板目录
CREATE_MAIL_SPOOL=yes

#修改默认值
[root@centos8 ~]# useradd -D -s /sbin/nologin
[root@centos8 ~]# cat /etc/default/useradd
# useradd defaults file
GROUP=100
HOME=/home
INACTIVE=-1
EXPIRE=
SHELL=/sbin/nologin
SKEL=/etc/skel
CREATE_MAIL_SPOOL=yes
#更多配置可以在帮助中Changing the default values小节查看
```

用户的配置相关文件

/etc/default/useradd

/etc/skel

/etc/login.defs

批量创建用户修改口令

```bash
#修改密码 
echo [密码] | passwd --stdin [用户名]    ##Ubuntu无法使用
#批量创建用户
newusers passwd 格式文件  
#批量修改密码 
echo username:passwd | chpasswd    #Ubuntu可以使用
```

### 3.2 usermod

usermod：修改用户账户属性

```bash
usermod [options] LOGIN
```

常用选项

- -u UID: 新UID
- -g GID: 新主组
- -G GROUP1[,GROUP2,...[,GROUPN]]]：新附加组，原来的附加组将会被覆盖；若保留原有，则要同时使 用-a 选项
- -s SHELL：新的默认SHELL
- -c 'COMMENT'：新的注释信息
- -d HOME: 新家目录不会自动创建；若要创建新 家目录并移动原家数据，同时使用-m选项
- -l login_name: 新的名字
- -L: lock指定用户,在/etc/shadow 密码栏的增加 !
- -U: unlock指定用户,将 /etc/shadow 密码栏的 ! 拿掉
- -e YYYY-MM-DD: 指明用户账号过期日期
- -f INACTIVE: 设定非活动期限，即宽限期

### 3.3 userdel

userdel：删除用户

```bash
userdel [options] LOGIN
```

常用选项

- -f, --force 强制
- -r, --remove 删除用户家目录和邮箱

### 3.4 id

id：查看用户相关的id信息

```bash
id [OPTION]... [USER]
```

常用选项

- -u: 显示UID
- -g: 显示GID
- -G: 显示用户所有所属的组的ID
- -n: 显示名称而不是数字，需配合ugG使用

### 3.5 su、getent

getent：getent - get entries from Name Service Switch libraries

```bash
getent [option]... database key...
```

su：命令可以切换用户身份，并且以指定用户的身份执行命令

```bash
su [options...] [-] [user [args...]]
```

常用选项

- -l --login su -l UserName 相当于 su - UserName
- -c, --command pass a single command to the shell with -c
- -s 使用指定的shell类型进行切换

加-和不加-的区别

 su UserName：非登录式切换，即不会读取目标用户的配置文件，不改变当前工作目录，即不完 全切换

 su - UserName：登录式切换，会读取目标用户的配置文件，切换至自已的家目录，即完全切换

 注意：su 切换新用户后，使用 exit 退回至旧的用户，而不要再用 su 切换至旧用户，否则会生成很多的 bash子进程，环境可能会混乱。

范例

```bash
#/sbin/nologin和/bin/false这两种shell类型都不能登陆
[root@Centos7 ~]# su user
[user@Centos7 root]$ pwd
/root
[user@Centos7 root]$ exit
exit

[root@Centos7 ~]# su - user
Last login: Mon Dec 21 02:49:38 CST 2020 on pts/0
[user@Centos7 ~]$ pwd
/home/user

[user@Centos7 root]$ getent passwd user
user:x:1000:1000:user:/home/user:/bin/bash
```

### 3.6 passwd

passwd：设置修改用户密码

```bash
passwd  [-k] [-l] [-u [-f]] [-d] [-e] [-n mindays] [-x maxdays] [-w warn‐
       days] [-i inactivedays] [-S] [--stdin] [username]
```

常用选项

- --stdin：从标准输入接收用户密码,Ubuntu无此选项
- -e：强制用户下次登录修改密码
- -d：删除指定用户密码
- -l：锁定指定用户
- -u：解锁指定用户

范例

```bash
#使用标准输出修改密码
[root@Centos7 ~]# echo 'qwe123' | passwd --stdin user

#限定用户下次登陆必须改密码
[root@Centos7 ~]# passwd -e user
Expiring password for user user.
passwd: Success
```

### 3.7 chage

chage：修改密码策略

```bash
chage [options] LOGIN
```

常见选项

- -d LAST_DAY #更改密码的时间(可以强制用户下次登陆必须改密码)
- -I --inactive INACTIVE #密码过期后的宽限期
- -E --expiredate EXPIRE_DATE #用户的有效期

### 3.8 用户相关的其它命令

chfn ：指定个人信息

```bash
chfn  [-f  full-name]  [-o office] ,RB [ -p office-phone] [-h home-phone]
       -u] [-v] [username]
       
[root@Centos7 ~]# chfn user
Changing finger information for user.
Name [user]: user
Office []: it
Office Phone []: 10000
Home Phone []: 111111
[root@Centos7 ~]# cat /etc/passwd
user:x:1000:1000:user,it,10000,111111:/home/user:/bin/bash
```

chsh：指定shell，相当于usermod -s

```bash
chsh [-s shell] [-l] [-u] [-v] [username]
[root@centos7 ~]#chsh -s /bin/csh user
[root@Centos7 ~]# chsh -s /bin/sh user
Changing shell for user.
Shell changed.
[root@Centos7 ~]# getent passwd user
user:x:1000:1000:user,it,10000,111111:/home/user:/bin/sh
```

finger：可看用户个人信息

```bash
chsh [username]
```

### 3.9 groupadd

groupadd：创建组

```bash
groupadd [options] group
```

常见选项

- -g 指定GID
- -r 创建系统组

范例

```bash
[root@Centos7 ~]# groupadd -r mysql
[root@Centos7 ~]# getent group mysql
mysql:x:995:
```

查看组

```bash
tail /etc/group
```

### 3.10 groupmod

groupmod：修改组属性

```bash
groupmod [OPTION]... group
```

常用选项

- -n group_name: 新名字
- -g GID: 新的GID

### 3.11 groupdel

groupdel：删除组

```bash
groupdel [options] GROUP
```

常见选项

- -f, --force 强制删除，即使是用户的主组也强制删除组

### 3.12 gpasswd

gpasswd：可以更改组密码，也可以修改附加组的成员关系

```bash
gpasswd [OPTION] GROUP
```

常见选项

- -a user 将user添加至指定组中
- -d user 从指定附加组中移除用户user
- -A user1,user2,... 设置有管理权限的用户列表

范例

```bash
#添加组成员
[root@Centos7 ~]# gpasswd -a user mysql
Adding user user to group mysql
[root@Centos7 ~]# groups user
user : user user1 mysql
#删除组成员
[root@Centos7 ~]# gpasswd -d user mysql
Removing user user from group mysql
```

### 3.13 groupmems

groupmems：管理**附加组**的成员关系

```bash
groupmems [options] [action]
```

常见选项

- -g, --group groupname #指定组 (只有root)
- -a, --add username #指定用户加入组
- -d, --delete username #从组中删除用户
- -p, --purge #从组中清除所有成员
- -l, --list #显示附加组成员列表

## 4.文件权限管理

### 4.1 chown

chown：修改文件的属主，也可以修改文件属组

```bash
chown [OPTION]... [OWNER][:[GROUP]] FILE...
chown [OPTION]... --reference=RFILE FILE...
#--reference=RFILE  参考指定的的属性，来修改  
#-R 递归，此选项不建议使用，非常危险！
```

范例

```bash
[root@Centos7 tmp]# ll
total 0
-rw-rw-r--. 1 user user 0 Dec 21 07:44 user.tx
[root@Centos7 tmp]# chown user2:user2 user.txt 
[root@Centos7 tmp]# ll
total 0
-rw-rw-r--. 1 user2 user2 0 Dec 21 07:44 user.tx

#f2复制f1的所属主，组
[17:54:31 root@centos8 data]#ll
total 0
-rw-r--r--. 1 test test 0 Jul 27 17:49 f1.txt
-rw-r--r--. 1 root root 0 Jul 27 17:54 f2.txt
[17:54:31 root@centos8 data]#chown --reference=f1.txt f2.txt 
[17:54:45 root@centos8 data]#ll
total 0
-rw-r--r--. 1 test test 0 Jul 27 17:49 f1.txt
-rw-r--r--. 1 test test 0 Jul 27 17:54 f2.txt

```

### 4.2 chgrp

chgrp：修改文件属组

```bash
chgrp [OPTION]... GROUP FILE...
chgrp [OPTION]... --reference=RFILE FILE...
```

范例

```bash
[root@centos8 data]# ll
total 0
-rw-rw-r-- 1 user user 0 Dec 21 11:43 user.txt
[root@centos8 data]# chgrp user1 user.txt 
[root@centos8 data]# ll
total 0
-rw-rw-r-- 1 user user1 0 Dec 21 11:43 user.txt
```

### 4.3 文件权限

**对文件的权限**

```bash
owner   属主，u
group   属组，g
other   其他，o
```

```bash
r (Readable)  可使用文件查看类工具，比如：cat，可以获取其内容
w (Writable)  可修改其内容
x (Excutable) 可以把此文件提请内核启动为一个进程，即可以执行（运行）此文件（此文件的内容必须是可执行）
```

**对目录的权限**

```bash
r 可以使用ls查看此目录中文件列表
w 可在此目录中创建文件，也可删除此目录中的文件，而和此被删除的文件的权限无关
x 可以cd进入此目录，可以使用ls -l查看此目录中文件元数据（须配合r权限），属于目录的可访问的最 小权限
X 只给目录x权限，不给无执行权限的文件x权限
```

![image-20230727174214146](C:\Users\NXTP\Desktop\Linux&Memo\image-20230727174214146.png)

```bash
前3所有者，中3所属组，后3其他
-rwxrwxrwx. 1 root root 0 Jul 27 17:49 f1.txt
-rw-r--r--. 1 root root 0 Jul 27 17:54 f2.txt
用户的最终权限，从左至右有序的逐条匹配，即所有者，所属组，其他。一旦匹配权限立即生效，不再向右查看其他权限。
```

### 4.4 chmod

chmod：更改文件权限

```bash
chmod [OPTION]... MODE[,MODE]... FILE...
chmod [OPTION]... OCTAL-MODE FILE...
chmod [OPTION]... --reference=RFILE FILE...
#MODE：who opt permission
#who:u,g,o,a (u所有者，g所属组，o其他，a全部)
#opt:+,-,=
#permission:r,w,x
```

范例

```bash
#两种方式修改权限
[root@centos8 data]# chmod o+w user.txt 
[root@centos8 data]# ll
total 0
-rw-rw-rw- 1 user user1 0 Dec 21 11:43 user.txt

[root@centos8 data]# chmod 755 user.txt 
[root@centos8 data]# ll
total 0
-rwxr-xr-x 1 user user1 0 Dec 21 11:43 user.txt
```

### 4.5 umask

umask：决定新建文件和文件夹的默认权限

实现方式

- 新建文件的默认权限: 666-umask，如果所得结果某位存在执行（奇数）权限，则将其权限+1,偶 数不变
- 新建目录的默认权限: 777-umask

非特权用户umask默认是 002

root的umask 默认是 022

范例

```bash
#查看umask
[root@centos8 data]# umask
0022
#修改umask
[root@centos8 data]#umask 002

#持久保存umask
全局设置： /etc/bashrc
用户设置：~/.bashrc

#(umask 666; touch 1111111111.txt) 0权限设置
----------. 1 root root  0 Jul 28 15:01 1111111111.txt

```

### 4.6 SUID、SGID、Sticky

**4.6.1 SUID**

前提：进程有属主和属组；文件有属主和属组

1.执行者对该程序拥有执行权限

- SUID只对二进制可执行程序有效，设置在目录上无意义
- 启动进程后，其进程属主为源程序文件属主
- 权限在执行过程中有效

范例

```bash
#两种修改方式
chmod u+s FILE...
chmod 6xxx FILE

[root@centos8 test]# chmod 6640 root.txt 
[root@centos8 test]# ll
-rwSr-S--- 1 root root 0 Dec 21 12:26 root.txt
```

**4.6.2 SGID**

二进制文件上的SGID权限功能：

- 启动为进程之后，其进程的属组为原程序文件的属组
- 任何一个可执行程序文件能不能启动为进程：取决发起者对程序文件是否拥有执行权限

目录上的SGID权限功能：

默认情况下，用户创建文件时，其属组为此用户所属的主组，一旦某目录被设定了SGID，则对此目录有 写权限的用户在此目录中创建的文件所属的组为此目录的属组，通常用于创建一个协作目录

SGID权限设定

```bash
#二进制文件设定权限
chmod g+s FILE...
chmod 2xxx FILE
[root@centos8 test]# chmod g+s root.txt 
-rw-r-S--- 1 root root 0 Dec 21 12:26 root.txt
[root@centos8 test]# chmod g-s root.txt 

#目录设定权限
chmod g+s DIR...
chmod 2xxx DIR
chmod g-s DIR...
```

**4.6.3 Sticky**

具有写权限的目录通常用户可以删除该目录中的任何文件，无论该文件的权限或拥有权 在目录设置Sticky 位，只有文件的所有者或root可以删除该文件

sticky 设置在文件上无意义

范例

```bash
#权限设定
chmod o+t DIR...
chmod 1xxx DIR
chmod o-t DIR...
[root@centos8 ~]#ll -d /tmp
drwxrwxrwt. 15 root root 4096 Dec 12 20:16 /tmp
```

### 4.7 chattr

chattr：设置文件的特殊属性，可以访问 root 用户误操作删除或修改文件

不能删除，改名，更改

```bash
chattr +i 
```

只能追加内容，不能删除，改名

```bash
chattr +a
```

显示特定属性

```bash
lsattr 
```

### 4.8 ACL

ACL：Access Control List，实现灵活的权限管理

```bash
setfacl 可以设置ACL权限
getfacl 可查看设置的ACL权限
```

范例

```bash
[root@centos8 test]# setfacl -m u:user:rw root.txt 
[root@centos8 test]# getfacl root.txt 
# file: root.txt
# owner: root
# group: root
user::rw-
user:user:rw-
group::r--
mask::rw-
other::---
#清楚所有ACL权限
[root@centos8 test]# setfacl -b root.txt
#复制file1的acl权限给file2
getfacl file1 | setfacl --set-file=-   file2  
```

# 5.VIM和正则表达式

## 1.VIM

### 1.1vim简介

vim是一款强大的文本编辑器，它和 vi 使用方法一致，但功能更为强大。官网：www.vim.org、中文手册：http://vimcdoc.sourceforge.net/

### 1.2使用vim

**1.2.1命令格式**

```bash
vim [options] [file ..]
```

常用选项

- +# 打开文件后，让光标处于第#行的行首，+默认行尾
- +/PATTERN 让光标处于第一个被PATTERN匹配到的行行首
- -d file1 file2… 比较多个文件，相当于 vimdiff

说明：当使用vim打开文件的时候，如果该文件存在，文件被打开并显示内容。如果该文件不存在，当编辑后第一次存盘时创建它

**1.2.2vim三种模式的切换**

三种模式分为

命令或普通模式、插入模式、扩展模式

- 命令模式 --> 插入模式

```bash
i insert, 在光标所在处输入
I 在当前光标所在行的行首输入
a append, 在光标所在处后面输入
A 在当前光标所在行的行尾输入
o 在当前光标所在行的下方打开一个新行
O 在当前光标所在行的上方打开一个新行
gg 到文件头部。
G 到文件尾部。
```

- 插入模式 --- ESC-----> 命令模式
- 命令模式 ---- : ----> 扩展命令模式
- 扩展命令模式 ----ESC,enter----> 命令模式

### 1.3扩展命令模式

##### 1.3.1 基本命令

```bash
w 		写（存）磁盘文件
wq 	写入并退出
x		写入并退出加密
q 		退出，如果修改了内容无法退出
q！   强制不存盘退出，即使更改都将丢失
r filename 读文件内容到当前文件中
!command 	执行命令
r!command 	读入命令的输出
```

##### 1.3.2 地址定界

格式

```bash
:start_pos,end_pos CMD
```

地址定界符

```bash
# 			#具体第#行，例如2表示第2行
#,# 		#从左侧#表示起始行，到右侧#表示结尾行
#,+# 		#从左侧#表示的起始行，加上右侧#表示的行数，范例：2,+3 表示2到5行
.   		#当前行
$ 			#最后一行
.,$-1 	#当前行到倒数第二行
% 			#全文, 相当于1,$

/pattern/   	#从当前行向下查找，直到匹配pattern的第一行,即:正则表达式
/pat1/,/pat2/ 	#从第一次被pat1模式匹配到的行开始，一直到第一次被pat2匹配到的行结束
#,/pat/     	#从指定行开始，一直找到第一个匹配patttern的行结束
/pat/,$     	#向下找到第一个匹配patttern的行到整个文件的结尾的所有行
```

地址定界后可跟一个编辑命令

```bash
d       	#删除
y 			#复制
w file 	#将范围内的行另存至指定文件中
r file 	#在指定位置插入指定文件中的所有内容
```

##### 1.3.3 查找和替换

```bash
s/要查找的内容/替换为的内容/修饰符
```

说明

```bash
要查找的内容：可使用基末正则表达式模式
替换为的内容：不能使用模式，但可以使用\1, \2, ...等后向引用符号；还可以使用“&”引用前面查找时查
找到的整个内容
```

修饰符：

```bash
i #忽略大小写
g #全局替换，默认情况下，每一行只替换第一次出现
gc #全局替换，每次替换前询问
```

查找替换中的分隔符/可替换为其它字符，如：#,@

```bash
s@/etc@/var@g
s#/boot#/#i
```

##### 1.3.4 定制vim

配置文件：

```bash
/etc/vimrc #全局
~/.vimrc #个人

#行号
显示：set number，简写 set nu
取消显示：set nonumber, 简写 set nonu

#忽略字符的大小写
启用：set ignorecase，简写 set ic
不忽略：set noic

#自动缩进
启用：set autoindent，简写 set ai
禁用：set noai

#设置光标所在行的标识线
启用：set cursorline，简写 set cul
禁用：set nocursorline

#Tab用指定空格的个数代替
启用：set tabstop=# 指定#个空格代替Tab
简写：set ts=4

#文件格式
启用windows格式：set fileformat=dos
启用unix格式：set fileformat=unix
简写 set ff=dos|unix

#Tab 用空格代替
启用：set expandtab 默认为8个空格代替Tab
禁用：set noexpandtab
简写：set et 

#语法高亮
启用：syntax on
禁用：syntax of

#高亮搜索
启用：set hlsearch
禁用：set nohlsearch

#显示Tab和换行符 ^I 和$显示
启用：set list
禁用：set nolist

#set showcmd
显示命令模式下输入的字符
```

### 1.4 命令模式

##### 1.4.1 退出VIM

ZZ | wq 保存退出

ZQ | q 不保存退出

##### 1.4.2 光标跳转

**字符间跳转：**

h: 左 l: 右 j: 下 k: 上

\#COMMAND：跳转由#指定的个数的字符

**单词间跳转：**

w：下一个单词的词首

e：当前或下一单词的词尾

b：当前或前一个单词的词首

\#COMMAND：由#指定一次跳转的单词数

**当前页跳转：**

H：页首 M：页中间行 L：页底

zt：将光标所在当前行移到屏幕顶端

zz：将光标所在当前行移到屏幕中间

zb：将光标所在当前行移到屏幕底端

**行首行尾跳转：**

^ 跳转至行首的第一个非空白字符

0 跳转至行首

$ 跳转至行尾

**行间移动：**

\#G 或者扩展命令模式下 :# 跳转至由第#行

G 最后一行

1G, gg 第一行

**命令模式翻屏操作**

Ctrl+f 向文件尾部翻一屏

Ctrl+b 向文件首部翻一屏

Ctrl+d 向文件尾部翻半屏

Ctrl+u 向文件首部翻半屏

##### 1.4.3 字符编辑

x 删除光标处的字符

\#x 删除光标处起始的#个字符

xp 交换光标所在处的字符及其后面字符的位置

~ 转换大小写

J 删除当前行后的换行符

##### 1.4.4 替换命令(replace)

r 只替换光标所在处的一个字符

R 切换成REPLACE模式（在末行出现-- REPLACE -- 提示）,按ESC回到命令模式

##### 1.4.5 删除命令（delete）

d 删除命令，可结合光标跳转字符，实现范围删除

d$ 删除到行尾

d^ 删除到非空行首

d0 删除到行首

dd： 剪切光标所在的行

\#dd 多行删除

D：从当前光标位置一直删除到行尾，等同于d$

dG  位置以下全部删除

:%d  删除所有

##### 1.4.6 复制命令(yank)

y 复制，行为相似于d命令

##### 1.4.7 粘贴命令(paste)

p 缓冲区存的如果为整行，则粘贴当前光标所在行的下方；否则，则粘贴至当前光标所在处的后面

P 缓冲区存的如果为整行，则粘贴当前光标所在行的上方；否则，则粘贴至当前光标所在处的前面

##### 1.4.8 改变命令(change)

c: 删除后切换成插入模式

c$

c^

c0

cb

##### 1.4.9 查找

/PATTERN：从当前光标所在处向文件尾部查找

?PATTERN：从当前光标所在处向文件首部查找

n：与命令同方向

N：与命令反方向

##### 1.4.10 撤消更改

u 撤销最近的更改，相当于windows中ctrl+z

\#u 撤销之前多次更改

U 撤消光标落在这行后所有此行的更改

Ctrl - r 重做最后的“撤消”更改，相当于windows中crtl+y

. 重复前一个操作

\#. 重复前一个操作#次

##### 1.4.11 高级用法

常见Command：y 复制、d 删除、gU 变大写、gu 变小写

范例

```bash
0y$ 命令
0   先到行头
y   从这里开始拷贝
$   拷贝到本行最后一个字符

#粘贴一个字符100次
100ihello [ESC] 
```

di" 光标在” “之间，则删除” “之间的内容

yi( 光标在()之间，则复制()之间的内容

vi[ 光标在[]之间，则选中[]之间的内容

dtx 删除字符直到遇见光标之后的第一个 x 字符

ytx 复制字符直到遇见光标之后的第一个 x 字符

### 1.5 可视化模式

在末行有”-- VISUAL -- “指示，表示在可视化模式

进入可视化模式按键

- v 面向字符，-- VISUAL --
- V 面向整行，-- VISUAL LINE --
- ctrl-v 面向块，-- VISUAL BLOCK --

范例：在文件行首插入#

```bash
输入ctrl+v 进入可视化模式
输入 G 跳到最后一行，选中每一行的第一个字符
输入 I 切换至插入模式
输入 #
按 ESC 键
```

### 1.6 多文件模式

vim FILE1 FILE2 FILE3 ...

:next 下一个

:prev 前一个

:first 第一个

:last 最后一个

:wall 保存所有

:qall 不保存退出所有

:wqall保存退出所有

### 1.7 多窗口模式

##### 1.7.1 多文件分割

vim -o|-O FILE1 FILE2 ...

-o: 水平或上下分割

-O: 垂直或左右分割（vim only）

在窗口间切换：Ctrl+w+方向键

##### 1.7.2 单文件窗口分割

Ctrl+w+s：split, 水平分割，上下分屏

Ctrl+w+v：vertical, 垂直分割，左右分屏

ctrl+w+q：取消相邻窗口

ctrl+w+o：取消全部窗口

:wqall 退出

### 1.8 vim寄存器

有26个命名寄存器和1个无命名寄存器，常存放不同的剪贴版内容，可以在同一个主机的不同会话（终端窗口）间共享

寄存器名称a，b,…,z,格式： ”寄存器 放在数字和命令之间

范例：

```bash
3"tyy 表示复制3行到t寄存器中 ，末行显示 3 lines yanked into "t
"tp 表示将t寄存器内容粘贴
```

### 1.9 标记和宏(macro)

ma 将当前位置标记为a，26个字母均可做标记， mb 、 mc 等等

'a 跳转到a标记的位置，实用的文档内标记方法，文档中跳跃编辑时很有用

qa 录制宏 a，a为宏的名称，末行提示： recording @a

q 停止录制宏

@a 执行宏 a

@@ 重新执行上次执行的宏

![image-20230731171310009](C:\Users\user\Desktop\Linux Memo\image-20230731171310009.png)

### 快捷键

| 快捷键            | 功能                                            |
| ----------------- | ----------------------------------------------- |
| `h`               | 光标向左移动一个字符                            |
| `j` 或 `Ctrl + J` | 光标向下移动一行                                |
| `k` 或 `Ctrl + P` | 光标向上移动一行                                |
| `l`               | 光标向右移动一个字符                            |
| `0`               | （数字 0）移动光标至本行开头                    |
| `$`               | 移动光标至本行末尾                              |
| `^`               | 移动光标至本行第一个非空字符处                  |
| `w`               | 向前移动一个词 （上一个字母和数字组成的词之后） |
| `W`               | 向前移动一个词 （以空格分隔的词）               |
| `5w`              | 向前移动五个词                                  |
| `b`               | 向后移动一个词 （下一个字母和数字组成的词之前） |
| `B`               | 向后移动一个词 （以空格分隔的词）               |
| `5b`              | 向后移动五个词                                  |
| `G`               | 移动至文件末尾                                  |
| `gg`              | 移动至文件开头                                  |

## 2 文本常见处理工具

### 2.1查看文件内容

##### 2.1.1 cat

cat 可以查看文本内容

```bash
cat [OPTION]... [FILE]...
```

常见选项

- -E：显示行结束符$
- -A：显示所有控制符
- -n：对显示出的每一行进行编号
- -b：非空行编号
- -s：压缩连续的空行成一行

##### 2.1.2 nl

nl：显示行号，相当于cat -b

```bash
nl [OPTION]... [FILE]...	
```

##### 2.1.3 tac

tac：逆向显示文本内容

```bash
tac [OPTION]... [FILE]...
```

##### 2.1.4 rev

rev：将同一行的内容逆向显示

```bash
rev [option] [file...]
[root@centos8 data]# rev lx.txt 
ba321
[root@centos8 data]# cat lx.txt 
123ab
```

##### 2.1.5 hexdump

hexdump：以十六进制，十进制，八进制或ASCII显示文件内容

```bash
hexdump [options] file...

-C 十六进制+ASCII显示
[root@centos8 data]# hexdump -C lx.txt 
00000000  31 32 33 61 62 0a                                 |123ab.|
00000006
[root@centos8 data]# cat lx.txt 
123ab
```

### 2.2分页查看文本内容

##### 2.2.1 more

more：分页查看文件（文本翻至尾部就会自动退出）

```bash
more [options] file...
```

常见选项

- -d 显示翻页以及退出提示

##### 2.2.2 less

less：分页查看文件（文本末页不会自动退出）

less 命令是man命令使用的分页器

```bash
[root@centos8 data]# cat passwd | less
```

### 2.3 显示文本前或后行内容

##### 2.3.1 head

head：显示文件或者标准输入前面内容

```bash
head [OPTION]... [FILE]...
```

常见选项

- -c # 指定获取前#字节
- -n # 指定获取前#行
- -# 同上

范例

```bash
[root@centos8 data]# head -2 lx.txt 
1
2
```

##### 2.3.2 tail

tail：和head 相反，查看文件或标准输入的倒数行

```bash
tail [OPTION]... [FILE]...
```

常用选项

- -c # 指定获取后#字节
- -n # 指定获取后#行
- -# 同上
- -f 跟踪显示文件fd新追加的内容,常用日志监控，相当于 --follow=descriptor,当文件删除再新建同名 文件,将无法继续跟踪文件
- -F 跟踪文件名，相当于--follow=name --retry，当文件删除再新建同名文件,将可以继续跟踪文件
- tailf 类似 tail –f，当文件不增长时并不访问文件

范例

```bash
[root@centos8 data]# tail -2 /data/lx.txt 
9
10
[root@centos8 data]# tail -fn0 /var/log/messages
#只查看最新发生的日志
[root@centos8 data]# tail -fn0 /data/lx.txt 
ab
```

### 2.4 cut

cut：提取文本文件或者STDIN数据指定列

```bash
cut OPTION... [FILE]...
```

常用选项

- -d DELIMITER: 指明分隔符，默认tab

- -f FILEDS:

  #: 第#个字段,例如:3

  #,#[,#]：离散的多个字段，例如:1,3,6

  #-#：连续的多个字段, 例如:1-6

  混合使用：1-3,7

- -c 按字符切割

- --output-delimiter=STRING指定输出分隔符

范例

```bash
#查看主机IP地址
[root@centos8 data]# ifconfig | head -2 | tail -1 | tr -s ' ' | cut -d' ' -f3
172.22.73.89
#显示df命令中磁盘使用率
[root@centos8 data]# df | tr -s ' ' | cut -d' ' -f5 | tr -dc '[0-9\n]'
0
0
[root@centos8 data]# df | tr -s ' ' % | cut -d% -f5 | tr -d [:alpha:]

0
0
```

### 2.5 paste

paste：合并多个文件同行号列到一行

```bash
paste [OPTION]... [FILE]...
```

常用选项

- -d 分隔符：指定分隔符，默认用TAB
- -s : 所有行合成一行显示

### 2.6 文本分析工具

##### 2.6.1 wc

wc：统计文件的行总数、单词总数、字节总数和字符总数

```bash
wc [OPTION]... [FILE]...
```

常用选项

- -l 只计数行数
- -w 只计数单词总数
- -L 显示文件中最长行的长度

```bash
[root@centos8 data]# wc -l lx.txt 
2 lx.txt
```

##### 2.6.2 sort

sort：把整理过的文本显示在STDOUT，不改变原始文件

```bash
sort [OPTION]... [FILE]...
```

常用选项

- -r 执行反方向（由上至下）整理
- -R 随机排序
- -n 执行按数字大小整理
- -u 选项（独特，unique），合并重复项，即去重
- -t c 选项使用c做为字段界定符
- -k# 指定排序的关键字

范例

```bash
#统计日志访问量
[root@centos8 data]#cut -d" " -f1 /var/log/nginx/access_log |sort -u|wc -l
201

[root@centos8 data]# cut -d: -f1,3 passwd | sort -k2 -nr | head -5
user3:2004
user2:2003
user1:1002
user:1001
unbound:996
```

##### 2.6.3 uniq

uniq：命令从输入中删除前后相接的重复的行

```bash
uniq [OPTION]... [INPUT [OUTPUT]]
```

常用选项

- -c: 显示每行重复出现的次数
- -d: 仅显示重复过的行
- -u: 仅显示不曾重复的行

uniq常和sort 命令一起配合使用：

范例

```bash
#统计日志访问量最多的请求
[root@centos8 data]# cut -d" " -f1 access_log | sort | uniq -c | sort -nr | head -1
2834 172.20.0.222

#取两个文件的相同行
[root@centos8 data]# cat a.txt b.txt | sort | uniq -d
200
#取两个文件的不同行
[root@centos8 data]# cat a.txt b.txt | sort | uniq -u
100
123
23
3321
34556
```

##### 2.6.4 diff

diff：令比较两个文件之间的区别

```bash
diff [OPTION]... FILES

[root@centos8 data]# cat a.txt b.txt 
a
b
c
a
b
bc
[root@centos8 data]# diff a.txt b.txt 
3c3
< c
---
> bc
```

## 3 正则表达式（Regular Expressions）

正则表达式分两类：

 基本正则表达式：BRE

 扩展正则表达式：ERE

 帮助：man 7 regex

### 3.1 基本正则表达式元字符

与通配符不同，通配 符功能是用来处理文件名，而正则表达式是处理文本内容中字符

##### 3.1.1 字符匹配

```bash
.   匹配任意单个字符，可以是一个汉字
[]   匹配指定范围内的任意单个字符，示例：[wang]   [0-9]   [a-z]   [a-zA-Z]
[^] 匹配指定范围外的任意单个字符,示例：[^wang]

[:alnum:] 字母和数字
[:alpha:] 代表任何英文大小写字符，亦即 A-Z, a-z
[:lower:] 小写字母,示例:[[:lower:]],相当于[a-z]
[:upper:] 大写字母
[:blank:] 空白字符（空格和制表符）
[:space:] 水平和垂直的空白字符（比[:blank:]包含的范围广）
[:cntrl:] 不可打印的控制字符（退格、删除、警铃...）
[:digit:] 十进制数字
[:xdigit:]十六进制数字
[:graph:] 可打印的非空白字符
[:print:] 可打印字符
[:punct:] 标点符号
```

##### 3.1.2 匹配次数

```bash
* 匹配前面的字符任意次，包括0次，贪婪模式：尽可能长的匹配
.* 任意长度的任意字符
\? 匹配其前面的字符0或1次,即:可有可无
\+ 匹配其前面的字符至少1次,即:肯定有，>=1
\{n\} 匹配前面的字符n次
\{m,n\} 匹配前面的字符至少m次，至多n次
\{,n\} 匹配前面的字符至多n次,<=n
\{n,\} 匹配前面的字符至少n次
```

##### 3.1.3 位置锚定

```bash
^ 行首锚 定，用于模式的最左侧
$ 行尾锚定，用于模式的最右侧
^PATTERN$ 用于模式匹配整行
^$ 空行
^[[:space:]]*$ 空白行
\< 或 \b 词首锚定，用于单词模式的左侧
\> 或 \b 词尾锚定，用于单词模式的右侧
\<PATTERN\> 匹配整个单词
```

##### 3.1.4 分组其它

**分组**

分组：() 将多个字符捆绑在一起，当作一个整体处理，如：(root)+ 后向引用：分组括号中的模式匹配到的内容会被正则表达式引擎记录于内部的变量中，这些变量的命名 方式为: \1, \2, \3, ...

\1 表示从左侧起第一个左括号以及与之匹配右括号之间的模式所匹配到的字符

**或者**

```bash
a\|b #a或b  
C\|cat #C或cat  
\(C\|c\)at #Cat或cat
```

### 3.2 扩展正则表达式

扩展正则于正则用法基本相同，大部分扩展正则相比正则只是取消了反斜线

##### 3.2.1 字符匹配元字符

```bash
. 任意单个字符
[wang] 指定范围的字符
[^wang] 不在指定范围的字符
[:alnum:] 字母和数字
[:alpha:] 代表任何英文大小写字符，亦即 A-Z, a-z
[:lower:] 小写字母,示例:[[:lower:]],相当于[a-z]
[:upper:] 大写字母
[:blank:] 空白字符（空格和制表符）
[:space:] 水平和垂直的空白字符（比[:blank:]包含的范围广）
[:cntrl:] 不可打印的控制字符（退格、删除、警铃...）
[:digit:] 十进制数字
[:xdigit:]十六进制数字
[:graph:] 可打印的非空白字符
[:print:] 可打印字符
[:punct:] 标点符号
```

##### 3.2.2 次数匹配

```bash
*   匹配前面字符任意次
? 0或1次
+ 1次或多次
{n} 匹配n次
{m,n} 至少m，至多n次
```

##### 3.2.3 位置锚定

```bash
^ 行首
$ 行尾
\<, \b 语首
\>, \b 语尾
```

##### 3.2.4 分组其它

分组：() 将多个字符捆绑在一起，当作一个整体处理，如：(root)+ 后向引用：分组括号中的模式匹配到的内容会被正则表达式引擎记录于内部的变量中，这些变量的命名 方式为: \1, \2, \3, ...

\1 表示从左侧起第一个左括号以及与之匹配右括号之间的模式所匹配到的字符

**或者**\

```bash
a\|b #a或b  
C\|cat #C或cat  
\(C\|c\)at #Cat或cat
```



### 3.2 扩展正则表达式

扩展正则于正则用法基本相同，大部分扩展正则相比正则只是取消了反斜线

##### 3.2.1 字符匹配元字符

```bash
. 任意单个字符
[wang] 指定范围的字符
[^wang] 不在指定范围的字符
[:alnum:] 字母和数字
[:alpha:] 代表任何英文大小写字符，亦即 A-Z, a-z
[:lower:] 小写字母,示例:[[:lower:]],相当于[a-z]
[:upper:] 大写字母
[:blank:] 空白字符（空格和制表符）
[:space:] 水平和垂直的空白字符（比[:blank:]包含的范围广）
[:cntrl:] 不可打印的控制字符（退格、删除、警铃...）
[:digit:] 十进制数字
[:xdigit:]十六进制数字
[:graph:] 可打印的非空白字符
[:print:] 可打印字符
[:punct:] 标点符号
```

##### 3.2.2 次数匹配

```bash
*   匹配前面字符任意次
?   0或1次
+   1次或多次
{n} 匹配n次
{m,n} 至少m，至多n次
```

##### 3.2.3 位置锚定

```bash
^ 行首
$ 行尾
\<, \b 语首
\>, \b 语尾
```

##### 3.2.4 分组其它

```bash
() 分组
后向引用：\1, \2, ...
| 或者
a|b #a或b
C|cat #C或cat
(C|c)at #Cat或cat
```



## 4 文本处理三剑客

### 4.1 grep

grep：文本搜索工具，根据用户指定的“模式”对目标文本逐行进行匹配检查；打印匹配到的行

```bash
grep [OPTIONS] PATTERN [FILE...]
```

常用选项

- -m# 匹配#次后停止

- -v 显示不被pattern匹配到的行
- -i 忽略字符大小写
- -n 显示匹配的行号
- -c 统计匹配的行数
- -q 不输出任何信息
- -f file 根据模式文件处理
- -E 使用ERE，相当于egrep
- -o 仅显示匹配到的字符串
- -e 实现多个选项间的逻辑or关系,如：grep –e ‘cat ’ -e ‘dog’ file
- -A# after 后#行
- -B# before 前#行
- -C# context 前后各#行

范例

```bash
[root@centos8 data]# grep "whoami" /etc/passwd
[root@centos8 data]# grep `whoami` /etc/passwd
root:x:0:0:root:/root:/bin/bash
operator:x:11:0:operator:/root:/sbin/nologin

#文件匹配
[root@centos8 data]# cat a.txt b.txt 
a
b
c
a
b
d
[root@centos8 data]# grep -f a.txt b.txt 
a
b

#查看磁盘最高使用率
[root@centos8 data]# df | grep ^/dev/vd | tr -s " " % | cut -d% -f5
7

#计算出所有人年龄之和
[root@centos8 data]# cat lx.txt 
xiaoming=20
xiaodong=18
xiaoqiang=22
[root@centos8 data]# cut -d= -f2 lx.txt | tr "\n" + | grep -Eo .*[0-9] | bc
60
[root@centos8 data]# grep -Eo [0-9]+ /data/lx.txt | tr "\n" + | grep -Eo .*[0-9] | bc
60
```

### 4.2 sed

sed 即 Stream EDitor.

##### 4.2.1 sed 工作原理

Sed是从文件或管道中读取一行，处理一行，输出一行，直到 最后一行。

##### 4.2.2 sed 基本用法

格式：

```bash
sed OPTIONS... [SCRIPT] [INPUTFILE...]
```

常用选项：

- -n 不输出模式空间内容到屏幕，即不自动打印
- -e 多点编辑
- -r, -E 使用扩展正则表达式
- -i.bak 备份文件并原处编辑
- -f /PATH/SCRIPT_FILE 从指定文件中读取编辑脚本

地址格式：

- 不给地址：对全文进行处理

- 单地址：

  \#：指定某一行

  $: 最后一行

  /pattern/: 被模式匹配的所有行

- 地址范围：

  addr1，addr2 从addr1到addr2行

  addr1,+addr2 从addr1到addr1+addr2行

  /part1/,/part2/ 模式1到模式2

- 步进：~

  1~2 奇数行

  2~2 偶数行

命令：

- p 打印当前模式空间内容，追加到默认输出之后
- Ip 忽略大小写输出
- d 删除模式空间匹配的行，并立即启用下一轮循环
- a [\]text 在指定行后面追加文本，支持使用\n实现多行追加
- i [\]text 在行前面插入文本
- c [\]text 替换行为单行或多行文本
- w /path/file 保存模式匹配的行至指定文件
- r /path/file 读取指定文件的文本至模式空间中匹配到的行后
- = 为模式空间中的行打印行号
- ! 模式空间中匹配行取反处理
- s/REGEXP/REPLACEMENT/修饰符 查找替换,支持使用其它分隔符，可以是其它形式：s@@@，s### 替换修饰符：
- g 行内全局替换
- p 显示替换成功的行
- w /PATH/FILE 将替换成功的行保存至文件中
- I,i 忽略大小写

范例：

```bash
[root@centos8 data]# ifconfig eth0 | sed -n '2p'
        inet 172.22.74.86  netmask 255.255.240.0  broadcast 172.22.79.255

[root@centos8 data]# sed -n '/root/p' passwd
root:x:0:0:root:/root:/bin/bash
operator:x:11:0:operator:/root:/sbin/nologin

[root@centos8 data]# cat seq.log 
1
2
3
4
5
6
7
8
9
10
[root@centos8 data]# sed -i.bak '2d;3d' seq.log
[root@centos8 data]# cat seq.log
1
4
5
6
7
8
9
10

[root@centos8 data]# sed -i '/abc/c\ 'qwe'' seq.log
[root@centos8 data]# cat seq.log
1
4
 qwe
6
7
8
9
10
[root@centos8 data]# sed -i 's/abc/def/' seq.log
[root@centos8 data]# cat seq.log
1
4
 qwe def
6
7
8
9
10
```

### 4.3 awk

##### 4.3.1 awk 工作原理和基本用法说明

用于格式化文本输出，有多个版本，GNU/Linux发布的AWK目前由自 由软件基金会（FSF）进行开发和维护，通常也称它为 GNU AWK。

gawk可以实现下面功能：

- 文本处理
- 格式化输出
- 执行算数运算和字符串操作

**gawk工作过程**：

[![gawk工作过程](https://www.cnblogs.com/bestvae/p/6.VIM%E5%92%8C%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F.assets/gawk%E5%B7%A5%E4%BD%9C%E8%BF%87%E7%A8%8B-1616842346281.jpg)](https://www.cnblogs.com/bestvae/p/6.VIM和正则表达式.assets/gawk工作过程-1616842346281.jpg)

BEGIN模块：

```bash
语法：
BEGIN	{action;… }
```

在awk开始从输入流中读取行之前被执行，只执行一次，这个模块是可选的

END模块：

```bash
语法：
END	{action;… }
```

在awk从输入流中读取完所有的行之后即被执行，只执行一次，这个模块也是可选的

格式：

```bash
gawk [ POSIX or GNU style options ] -f program-file  file ...
gawk [ POSIX or GNU style options ]  program-text  file ...
```

常用选项：

- -F fs 指明fs作为输入时的分隔字符，默认分隔符是若干个连续空白符
- -v var=value 变量赋值

**program-text格式：**

```bash
‘pattern   { action statements }’
pattern   { ‘action statements’}
```

注意：这里的单引号很有必要，如果没有会报语法错误

pattern：决定动作语句何时触发及触发事件,而且当pattern的值为真才会触发后续动作。

pattern常见类型：

- BEGIN
- END
- 正则表达式

action statements：对数据进行什么处理，放在{}内指明

action statements常见类型：

- output statements：print,printf
- Expressions：算术，比较表达式等
- Compound statements：组合语句
- Control statements：if, while等
- input statements

**域和记录**：

- 每一条读入记录会由分隔符分隔为一个个字段，这些字段称为域(field)，用$1，$2，...，$n来标识域，$0则表示所有域
- 文件的每一行称为记录record
- 如果省略action，则默认执行 print $0 的操作

##### 4.3.2 动作 print

格式：

```bash
print item1, item2, …
```

说明：

- 逗号作为分隔符
- 输出item可以字符串，也可是数值；当前记录的字段、变量或awk的表达式
- 如省略item，相当于print $0
- 固定字符符需要用" " 引起来，而变量和数字不需要

范例：

```bash
[root@centos8 ~]$awk 'BEGIN{print "hello"}'
hello

[root@centos8 ~]$awk -F: '{print $1,$3}' /etc/passwd
root 0
bin 1
daemon 2
adm 3
lp 4
sync 5
```

范例：取出网站访问量最大的前3个IP

```bash
[root@centos8 data]$awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -3
     18 60.191.36.75
      6 209.141.33.74
      2 74.120.14.37
```

范例：取出分区利用率

```bash
[root@centos8 data]$df | grep "/dev/vd" | awk -F' +|%' '{print $5}'
7
```

请提取”.print.com”前面的主机名部分并写入到回到该文件中

```bash
请提取”.print.com”前面的主机名部分并写入到回到该文件中
[root@centos8 ~]#cat host_list.log
1 www.print.com
2 blog.print.com
3 study.print.com
4 linux.print.com
5 python.print.com

[root@centos8 ~]$awk -F' +|.' '{print $2}' host_list.log 
#上面这种写法会把.当作正则表达式，所以不能输出结果
[root@centos8 ~]#awk -F"[ .]" '{print $2}' host_list.log
www
blog
study
linux
python
[root@centos8 ~]#awk -F"[ .]" '{print $2}' host_list.log >> host_list.log
```

##### 4.3.3 awk变量

awk中的变量分为：内置和自定义变量

常见内置变量：

- FS：输入字段分隔符，默认为空白字符,功能相当于 -F
- OFS：输出字段分隔符，默认为空白字符
- RS：输入记录record分隔符，指定输入时的换行符
- ORS：输出记录分隔符，输出时用指定符号代替换行符
- NF：域数量
- NR：记录的编号
- FILENAME：当前文件名
- ARGC：命令行参数的个数
- ARGV：数组，保存的是命令行所给定的各参数，每一个参数：ARGV[0]，......

范例：

```bash
#FS
[root@centos8 data]$awk -v FS=' |*' '{print $2}' host_list.log 
www
blog
study
root@centos8 data]$awk -F' |*' '{print $2}' host_list.log 
www
blog
study

#OFS
[root@centos8 ~]$awk -v FS=':' -v OFS=':' '{print $1,$3,$7}' /etc/passwd | head -n1
root:0:/bin/bash

#RS
[root@centos8 data]$cat host_list.log 
1 www*print*com
[root@centos8 data]$awk -v RS=' ' '{print }' host_list.log
1
www*print*com

#ORS
[root@centos8 data]$awk -v RS=' ' -v ORS='###' '{print }' host_list.log
1###www*print*com
###

#NF
[root@centos8 data]$cat host_list.log 
1 www*print*com
[root@centos8 data]$awk '{print NF}' host_list.log 
2
[root@centos8 data]$awk '{print $(NF-1)}' host_list.log 
1

#NR
[root@centos8 data]$awk '{print NR,$2}' host_list.log 
1 www*print*com
[root@centos8 data]$ifconfig eth0 | awk 'NR==2{print $2}'
172.22.74.86

#FNR
[root@centos8 ~]#awk '{print NR,$0}' /etc/issue /etc/redhat-release
1 \S
2 Kernel \r on an \m
3
4 CentOS Linux release 8.0.1905 (Core)
[root@centos8 script40]#awk '{print FNR,$0}' /etc/issue /etc/redhat-release
1 \S
2 Kernel \r on an \m
3
1 CentOS Linux release 8.0.1905 (Core)

#FILENAME
[root@centos8 data]$awk '{print FILENAME}' host_list.log 
host_list.log

#ARGC
[root@centos8 data]$awk '{print ARGC}' /etc/issue 
2
2
2
第一个参数是awk，第二个是/etc/issue
```

自定义变量：

- -v var=value
- 在program中直接定义

```bash
[root@centos8 data]$awk -v name='tom cat' BEGIN'{print name}'
tom cat
[root@centos8 data]$awk 'BEGIN{name="tomcat";print name}'
tomcat
```

##### 4.3.4 printf

与print不同的是printf可以自定义输出格式

格式：

```bash
printf “FORMAT”, item1, item2, ...
```

说明：

- 必须指定FORMAT
- 不会自动换行，需要显式给出换行控制符 \n
- FORMAT中需要分别为后面每个item指定格式符

格式符：

- %c：显示字符的ASCII码
- %d, %i：显示十进制整数
- %e, %E：显示科学计数法数值
- %f：显示为浮点数
- %g, %G：以科学计数法或浮点形式显示数值
- %s：显示字符串
- %u：无符号整数
- %%：显示%自身

注意：格式符应与item一一对应

修饰符

```bash
#[.#] 第一个数字控制显示的宽度；第二个#表示小数点后精度，如：%3.1f
- 左对齐（默认右对齐） 如：%-15s
+   显示数值的正负符号   如：%+d
```

范例：

```bash
[root@centos8 data]$awk -F: '{printf "%s",$1}' /etc/passwd
rootbindaemonadmlpsyncshutdownhaltmailoperatorgamesftpnobodydbussystemd-coredumpsystemd-resolvetsspolkitdlibstoragemgmtunboundsetroubleshootcockpit-wscockpit
[root@centos8 data]$awk -F: '{printf "%s\n",$1}' /etc/passwd
root
bin
......

[root@centos8 data]$awk -F: '{printf "%-20s %d\n",$1,$3}' /etc/passwd
root                 0
bin                  1
daemon               2
```

##### 4.3.5 操作符

算数操作符：

```bash
x+y, x-y, x*y, x/y, x^y, x%y
-x：转换为负数
+x：将字符串转换为数值
```

赋值操作符：

```bash
=, +=, -=, *=, /=, %=, ^=，++, --
```

比较操作符：

```bash
==, !=, >, >=, <, <=
```

模式匹配符：

```bash
~ 左边是否和右边匹配，包含关系
!~ 是否不匹配
```

逻辑操作符：

```bash
与：&&，并且关系
或：||，或者关系
非：!，取反
```

条件表达式（三目表达式）

```bash
selector?if-true-expression:if-false-expression
```

范例：

```bash
#先加和后加
[root@centos8 data]$awk 'BEGIN{i=0;print i++,i}'
0 1
[root@centos8 data]$awk 'BEGIN{i=0;print ++i,i}'
1 1

[root@centos8 data]$awk -v n=1 '!n++{print n}' /etc/passwd
[root@centos8 data]$awk -v n=0 '!n++{print n}' /etc/passwd
1

#模式匹配
[root@centos8 data]$awk -F: '$0 ~ /root/{print $1}' /etc/passwd
root
operator

#变量取反值
[root@centos8 data]$awk 'BEGIN{print i}'

[root@centos8 data]$awk 'BEGIN{print !i}'
1
[root@centos8 data]$awk 'BEGIN{i="ab";print !i}'
0
[root@centos8 data]$awk -v i=0 'BEGIN{print !i}'
1
[root@centos8 data]$awk -v i='' 'BEGIN{print !i}'
1

#逻辑判断
awk -F:   '$3>=0 && $3<=1000 {print $1,$3}' /etc/passwd
```

##### 4.3.6 关于PATTERN

根据pattern条件，过滤匹配的行，再做处理

格式：

```bash
pattern { action statements }’
```

常用类型：

- 未指定：空模式，匹配每一行
- /regular expression/：仅处理能够模式匹配到的行，需要用/ /括起来，而且使用扩展正则
- relational expression: 关系表达式，结果为“真”才会被处理
- /pat1/,/pat2/：从part1开始匹配到part2
- BEGIN/END模式

范例：

```bash
#空模式
[root@centos8 data]$awk '{print $1}' /etc/redhat-release 
CentOS

#/regular expression/
[root@centos8 data]$awk '/UUID/{print $1}' /etc/fstab 
UUID=ccd25378-c82e-4bea-ad12-81fba73fdf70

[root@centos8 data]$awk '0' /etc/redhat-release 
[root@centos8 data]$awk '1' /etc/redhat-release 
CentOS Linux release 8.2.2004 (Core)

#relational expression
[root@centos8 data]$awk 'NR>=3 && NR<=6{print $1}' /etc/passwd
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync

#/pat1/,/pat2/
[root@centos8 data]$awk '/^root/,/^daemon/' /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
```

##### 4.3.7 条件判断 if-else

格式：

```bash
if(condition){statement;…}[else statement]
if(condition1){statement1}else if(condition2){statement2}else if(condition3)
{statement3}......else{statementN}
```

范例：

```bash
[root@centos8 data]$awk -F: '{if($3>=1000)print $1,$3}' /etc/passwd
nobody 65534
git 1000
mageia 1100
slackware 2002
user 2003

[root@centos8 data]$awk -F: '{if($3>=1000){print "common user",$1,$3} else {print "sysuser",$1,$3}}' /etc/passwd
sysuser root 0
sysuser bin 1

[root@centos8 data]$awk -F: '{if($3>=1000)print "common user",$1,$3; else print "sysuser",$1,$3}' /etc/passwd
sysuser root 0
sysuser bin 1
```

##### 4.3.8 switch语句

格式：

```bash
switch(expression) {case VALUE1 or /REGEXP/: statement1; case VALUE2 or
/REGEXP2/: statement2; ...; default: statementn}
```

##### 4.3.9 循环while

格式：

```bash
while (condition) {statement;…}
```

使用场景： 对一行内的多个字段逐一类似处理时使用 ，对数组中的各元素逐一处理时使用

```bash
[root@centos8 data]$awk '{i=1;while(i<=NF){print $i,length($i); i++}}' /data/host_list.log
1 1
www*print*com 13
linux 5
new 3
test1 5
```

##### 4.3.10 循环 do-while

格式：

```bash
do {statement;…}while(condition)
```

意义：无论真假，至少执行一次循环体

##### 4.3.11 循环for

格式：

```bash
for(expr1;expr2;expr3) {statement;…}
```

可以用来遍历数组中的元素

```bash
for(var in array) {for-body}
```

范例：

```bash
#求1-100以内数字之和
[root@centos8 data]$awk 'BEGIN{sum=0;for(i=1;i<=100;i++){sum+=i};print sum}'
5050

time (total=0;for i in {1..10000};do total=$(($total+i));done;echo $total)
time (for ((i=0;i<=10000;i++));do let total+=i;done;echo $total)
time (seq –s ”+” 10000|bc)
```

相比较之下awk计算的速度最快，性能最好

##### 4.3.12 continue和break

continue 中断本次循环

break 中断整个循环

范例：

```bash
[root@centos8 ~]#awk 'BEGIN{sum=0;for(i=1;i<=100;i++)
{if(i%2==0)continue;sum+=i}print sum}'
2500
```

##### 4.3.13 next

可以提前结束对本行处理而直接进入下一行处理（awk自身循环）

用法与continue和break相同

##### 4.3.14 数组

awk的数组为关联数组

格式：

```bash
array_name[index-expression]
```

说明：

- 可使用任意字符串；字符串要使用双引号括起来
- 如果某数组元素事先不存在，在引用时，awk会自动创建此元素，并将其值初始化为“空串”
- 若要判断数组中是否存在某元素，要使用“index in array”格式进行遍历

范例：

```bash
#基本用法
[root@centos8 data]$awk 'BEGIN{weekdays["mon"]="Monday";weekdays["tue"]="Tuesday";print weekdays["mon"]}'
Monday

#去重
[root@centos8 data]$cat file 
a
b
a
bb
b
d
[root@centos8 data]$awk '!line[$0]++' file 
a
b
bb
d

#判断索引是否存在
[root@centos8 data]$awk 'BEGIN{weekdays["mon"]="Monday";weekdays["tue"]="Tuesday";if ("mon" in weekdays){print "存在"}else{print "不存在"}}'
存在
[root@centos8 data]$awk 'BEGIN{weekdays["mon"]="Monday";weekdays["tue"]="Tuesday";if ("awk" in weekdays){print "存在"}else{print "不存在"}}'
不存在
```

使用for循环遍历数组

```bash
for(var in array) {for-body}
[root@centos8 data]$awk 'BEGIN{weekdays["mon"]="Monday";weekdays["tue"]="Tuesday";for(i in weekdays){print weekdays[i]}}'
Tuesday
Monday
```

##### 4.3.15 awk函数

常见内置函数:

- rand()：返回0和1之间一个随机数
- srand()：配合rand() 函数,生成随机数的种子
- int()：返回整数

自定义函数:

格式：

```bash
#定义
function name ( parameter, parameter, ... ) {
   statements
   return expression
}

#调用
pattern{action name ( parameter, parameter, ... )}
```

##### 4.3.16 awk脚本

将awk程序写成脚本，直接调用或执行

范例：

```bash
#使用-f选项调用
[root@centos8 data]$awk -F: -f passwd.awk /etc/passwd
nobody 65534
git 1000
mageia 1100
slackware 2002
user 2003
[root@centos8 data]$cat passwd.awk 
{if($3>=1000)print $1,$3}

#awk脚本调用
[root@centos8 data]$cat passwd.awk 
#!/bin/awk -f
{if($3>=1000)print $1,$3}
[root@centos8 data]$chmod +x passwd.awk 
[root@centos8 data]$./passwd.awk -F: /etc/passwd
nobody 65534
git 1000
mageia 1100
slackware 2002
user 2003
```

向awk脚本传递参数

格式：

```bash
awkfile var=value var2=value2... Inputfile
```

注意：在BEGIN过程中不可用。直到首行输入完成以后，变量才可用。可以通过-v 参数，让awk在执行 BEGIN之前得到变量的值。命令行中每一个指定的变量都需要一个-v参数

练习

1.使用ifconfig获取本机的IPV4地址

```bash
grep broadcast lx.t | sed -nr 's/^[^0-9]+([0-9.]+).*$/\1/p'
```

2.删除/etc/fstab文件种所有以#开头，后面至少跟一个空白字符的行的行首的#和空白字符

```bash
sed -ri.bak 's/^#[[:blank:]]+//' fstab
```

3.处理/etc/fstab路径，使用sed命令取出其目录名和基名

```bash
[root@centos8 data]# echo /etc/fstab | sed -r 's#(.*)/([^/]+/?)#\1#'
/etc
[root@centos8 data]# echo /etc/fstab | sed -r 's#(.*)/([^/]+/?)#\2#'
fstab
```

4.解决Dos攻击生产案例：根据web日志或者或者网络连接数，监控当某个IP并发连接数或者短时内 PV达到100，即调用防火墙命令封掉对应的IP，监控频率每隔5分钟。防火墙命令为：iptables -A INPUT -s IP -j REJECT

```bash
#方式1
[root@centos8 script]$cat iptable.sh 
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -nr | while read times ip;do 
	if [ $times -gt 5 ];then
    	iptables -A INPUT -s $ip -j REJECT
    fi
done
[root@centos8 script]$cat !*
cat /var/spool/cron/root
*/5 * * * * /data/script/iptable.sh

#方式2
[root@centos8 script]$cat iptable.sh 
awk '{count[$1]++}END{for(i in count){if(count[i]>5){system("iptables -A INPUT -s "$ip" -j REJECT")}}}' /var/log/nginx/access.log
[root@centos8 script]$cat !*
cat /var/spool/cron/root
*/5 * * * * /data/script/iptable.sh
```

# 6.shell脚本编程

## 1.shell 脚本语言的基本用法

### 1.1shell 脚本创建

1.格式要求：首行shebang机制

```bash
#!/bin/bash
#!/usr/bin/python
#!/usr/bin/perl 
```

2.添加执行权限，在命令行上指定脚本的绝对或者相对路径，也可以运行脚本解释器直接运行脚本

### 1.2脚本的注释规范

1、第一行一般为调用使用的语言

2、程序名，避免更改文件名为无法找到正确的文件

3、版本号

4、更改后的时间

5、作者相关信息

6、该程序的作用，及注意事项

7、最后是各版本的更新简要说明

### 1.3脚本的执行方式

```bash
[root@centos8 data]# cat shell.sh 
#!/bin/bash
ls
[root@centos8 data]# chmod +x shell.sh 
#相对路径执行
[root@centos8 data]# ./shell.sh 
a.txt  b.txt  linxu.txt  message  passwd  shell.sh  systeminfo.sh  test  vimrc
#bash执行
[root@centos8 data]# bash shell.sh 
a.txt  b.txt  linxu.txt  message  passwd  shell.sh  systeminfo.sh  test  vimrc
#执行远程主机的shell脚本
[root@centos8 data]# #curl -s http://10.0.0.8/hello.sh|bash
[root@centos8 ~]#wget -qO - http://www.wangxiaochun.com/testdir/hello.sh |bash
```

### 1.4shell脚本调试

脚本常见的3种错误：

- 语法错误，后续的命令不继续执行，可以用bash -n检查错误，提示的出错行数不一定是准确的
- 命令错误，默认后续的命令还会继续执行，用bash -n 无法检查出来 ，可以使用 bash -x 进行观察
- 逻辑错误，只能使用 bash -x 进行观察

**bash命令**

bash -n：只检测脚本中的语法错误，但无法检查出命令错误，但不真正执行脚本

bash -x：调试并执行脚本

### 1.5变量

变量表示命名的内存空间，将数据放在内存空间中，通过变量名引用，获取数据

##### 1.5.1 shell变量类型

内置变量，如：PS1，PATH，UID，HOSTNAME，$$，BASHPID，PPID，$?，HISTSIZE

用户自定义变量

##### 1.5.2 变量的定义和引用

- 普通变量：生效范围为当前shell进程；对当前shell之外的其它shell进程，包括当前shell的子shell 进程均无效
- 环境变量：生效范围为当前shell进程及其子进程
- 本地变量：生效范围为当前shell进程中某代码片断，通常指函数

**变量赋值**

```bash
#变量的赋值可以是以下多种形式
直接字串：name='root'
变量引用：name="$USER"
命令引用：name=`COMMAND` 或者 name=$(COMMAND)  例：FILE=`ls /tmp`
```

注意：变量赋值是临时生效，当退出终端后，变量会自动删除，无法持久保存

**变量引用**

```bash
$name
${name}
```

范例

```bash
[root@centos8 data]# name='xiaoming'
[root@centos8 data]# echo $name
xiaoming

[root@centos8 data]# user='yonghu'
[root@centos8 data]# name="$user"
[root@centos8 data]# echo $name
yonghu

[root@centos8 data]# name=`ls -l /data/`
[root@centos8 data]# echo $name
total 8 -rw-r--r-- 1 root root 403 Jan 2 17:39 systeminfo.sh -rw-r--r-- 1 root root 110 Jan 3 10:17 test
[root@centos8 data]# echo "$name"
total 8
-rw-r--r-- 1 root root 403 Jan  2 17:39 systeminfo.sh
-rw-r--r-- 1 root root 110 Jan  3 10:17 test


[12:03:58 root@centos8 run]#echo $NUM
1 2 3 4 5 6 7 8 9 10
[12:04:02 root@centos8 run]#echo "$NUM"
1
2
3
4
5
6
7
8
9
10


```

范例：变量追加值

```bash
[root@centos8 data]# echo $user
yonghu
[root@centos8 data]# user+=:xiaoming
[root@centos8 data]# echo $user
yonghu:xiaoming
```

**显示和删除变量**

```bash
#set查看所有变量
#unset <name>删除指定变量
#使用上述方式设置变量是临时保存于内存当中，当退出终端就会销毁
```

##### 1.5.3 环境变量

**说明**

- 子进程（包括孙子进程）可以继承父进程的变量，但是无法让父进程使用子进程的变量
- 一旦子进程修改从父进程继承的变量，将会新的值传递给孙子进程
- 一般只在系统配置文件中使用，在脚本中较少使用

**环境变量的声明和赋值**：

```bash
#声明并赋值
export name=VALUE
declare -x name=VALUE

#或者分两步实现
name=VALUE
export name
```

**变量引用：**

```bash
$name
${name}
```

**显示所有环境变量：**

```bash
env
printenv
export
declare -x	
```

**删除变量**

```bash
unset <name>			
```

bash的内建环境变量

```bash
PATH
SHELL
USER
UID
HOME
PWD
SHLVL #shell的嵌套层数，即深度
LANG
MAIL
HOSTNAME
HISTSIZE
_   #下划线 表示前一命令的最后一个参数
```

##### 1.5.4 只读变量

只能声明定义，但后续不能修改和删除，即常量；退出当前终端可以销毁声明的只读变量

声明只读变量：

```bash
readonly name
declare -r name
```

查看只读变量

```bash
readonly [-p]
declare -r
```

将该目录加入`PATH`环境变量之后，就可以在虚拟目录结构的**任意位置**执行这个程序了(暂时生效)。

```bash
$ ls /home/christine/Scripts/
myprog
$ echo $PATH
/home/christine/.local/bin:/home/christine/bin:/usr/local/bin:/usr/
bin:/usr/local/sbin:/usr/sbin
$
$ PATH=$PATH:/home/christine/Scripts
$
$ myprog
The factorial of 5 is 120
```



##### 1.5.5 变量位置

在bash shell中内置的变量, 在脚本代码中调用通过命令行传递给脚本的参数

```bash
$1, $2, ... 对应第1个、第2个等参数，shift [n]换位置
$0 命令本身,包括路径
$* 传递给脚本的所有参数，全部参数合为一个字符串
$@ 传递给脚本的所有参数，每个参数为独立字符串
$# 传递给脚本的参数的个数
#清空所有位置变量
set --
```

范例

```bash
[root@centos8 data]# cat arg.sh 
#!/bin/bash
echo "arg1 is $1"
echo "arg1 is $2"
echo "arg1 is $3"
echo "the number of arg is $#"
echo "all args is $*"
echo "all args is $@"
echo "the script name is `basename $0`"

[root@centos8 data]# bash arg.sh {1..5}
arg1 is 1
arg1 is 2
arg1 is 3
the number of arg is 5
all args is 1 2 3 4 5
all args is 1 2 3 4 5
the script name is arg.sh
```

**rm命令修改**

```bash
#用户使用rm命令时不删除文件而是移动到/tmp文件夹
[root@centos8 data]# cat rm.sh 
COLOR="echo -e \E[1;31m"
END="\E[0m"
DIR=/tmp/`date +%F_%H-%M-%S`
mkdir $DIR
mv $* $DIR
$COLOR move $* to $DIR $END

[16:36:09 root@centos8 ~]#alias rm
alias rm='/root/rm.sh'
```

##### 1.5.6 退出状态码

$?

$?的值为0，代表成功

$?的值是1到255 #代表失败

注意：

- 如果未给脚本指定退出状态码，整个脚本的退出状态码取决于脚本中执行的最后一条命令的状态码

##### 1.5.7 脚本安全和 set

**$- 变量**

```bash
[root@centos8 data]# echo $-
himBHs
```

h：hashall，打开选项后，Shell 会将命令所在的路径hash下来，避免每次都要查询。通过set +h将h选 项关闭

i：interactive-comments，包含这个选项说明当前的 shell 是一个交互式的 shell。所谓的交互式shell, 在脚本中，i选项是关闭的 m：monitor，打开监控模式，就可以通过Job control来控制进程的停止、继续，后台或者前台执行等

B：braceexpand，大括号扩展

H：history，H选项打开，可以展开历史列表中的命令，可以通过!感叹号来完成，例如“!!”返回上最近的 一个历史命令，“!n”返回第 n 个历史命令

**set 命令**

-u 在扩展一个没有设置的变量时，显示错误信息， 等同set -o nounset

-e 如果一个命令返回一个非0退出状态值(失败)就退出， 等同set -o errexit

-o option 显示，打开或者关闭选项

-  显示选项：set -o
-  打开选项：set -o 选项
-  关闭选项：set +o 选项

### 1.6 printf

printf：格式化要输出的数据

```bash
printf FORMAT [ARGUMENT]...
```

常用格式替换符

- %s 字符串
- %f 浮点格式
- %b 相对应的参数中包含转义字符时，可以使用此替换符进行替换，对应的转义字符会被转 义
- %c ASCII字符，即显示对应参数的第一个字符
- %d,%i 十进制整数
- %o 八进制值
- %u 不带正负号的十进制值
- %x 十六进制值（a-f）
- %X 十六进制值（A-F）
- %% 表示%本身
- %s 中的数字代表此替换符中的输出字符宽度，不足补空格，默认是右对齐，%-10s表示10个字 符宽，- 表示左对齐

常用转义字符

- \a 警告字符，通常为ASCII的BEL字符
- \n 换行
- \r 回车
- \t 水平制表符
- \v 垂直制表符

范例

```bash
[root@centos8 data]# printf "%d\n" 123 321 123
123
321
123

[root@centos8 data]# printf "(%s) " 1 2 3
(1) (2) (3) [root@centos8 data]# 
[root@centos8 data]# printf "(%s) " 1 2 3;echo 
(1) (2) (3) 
[root@centos8 data]# printf "(%s) " 1 2 3;echo ""
(1) (2) (3) 


[root@centos8 data]# printf "%10s" name;echo
```

### 1.7算术运算

shell 支持算术运算，但只支持整数，不支持小数

shell运算符

```bash
+
-
*
/
% 取模，即取余数，示例：9%4=1，5%3=2
** 乘方
```

实现算数运算的方式

```bash
1.	let var=算数表达式
2.	（（var=算数表达式））
3.	var=$[算数表达式]
4.	var=$((算数表达式))
5.	delare -i var = 数值
6. echo '算术表达式' | bc 
7.	var=$(expr arg1 arg2 arg3 ...)
```

范例

```bash
#生成 1 - 50 之间随机数
[root@centos8 data]# echo $[RANDOM%50+1]
12

#随机颜色
[09:53:45 root@centos8 ~]#echo -e "\E[1;$[RANDOM%7+31]mhello\e[0m"
```

增强型赋值：

```bash
+= i+=10 相当于 i=i+10
-= i-=j   相当于 i=i-j
*=
/=
%=
++ i++,++i   相当于 i=i+1
-- i--,--i   相当于 i=i-1
```

范例

```bash
#i++和++i
[root@centos8 ~]#unset i j ; i=1; let j=i++; echo "i=$i,j=$j"
i=2,j=1
[root@centos8 ~]#unset i j ; i=1; let j=++i; echo "i=$i,j=$j"
i=2,j=2

#使用计算器实现浮点运算
[root@centos8 data]# echo "scale=3;20/3" | bc
6.666
```

### 1.8逻辑运算

与：&：和0相与，结果为0，和1相与，结果保留原值

```bash
1 与 1 = 1 
1 与 0 = 0 
0 与 1 = 0 
0 与 0 = 0
```

或：|：和1相或结果为1，和0相或，结果保留原值

```bash
 1 或 1 = 1
 1 或 0 = 1
 0 或 1 = 1
 0 或 0 = 0 
```

非：！

```bash
 ! 1 = 0    ! true
 ! 0 = 1    ! false
```

异或：^

相同为假，不同为真。

```bash
1 ^ 1 = 0
1 ^ 0 = 1
0 ^ 1 = 1
0 ^ 0 = 0 
```

范例

```bash
#注意区分逻辑运算的二进制值与$?的返回值含义刚好相反
#利用异或可以实现两个变量的数值交换
[root@centos8 data]# x=30;y=20;x=$[x^y];y=$[x^y];x=$[y^x];echo "x=$x y=$y"
x=20 y=30
```

短路运算

- 短路与

  CMD1 短路与 CMD2

  第一个CMD1结果为假 (0)，总的结果必定为0，因此不需要执行CMD2

- 短路或

  CMD1 短路或 CMD2

  第一个CMD1结果为真 (1)，总的结果必定为1，因此不需要执行CMD2

### 1.9条件测试命令

若真，则状态码变量 $? 返回0

若假，则状态码变量 $? 返回1

```bash
[root@centos8 ~]#unset x
[root@centos8 ~]#test -v x
[root@centos8 ~]#echo $?
1
```

```bash
[root@centos8 ~]#x=10
[root@centos8 ~]#test -v x
[root@centos8 ~]#echo $?
0
```

条件测试命令：

- test EXPRESSION
- [ EXPRESSION ] #和test 等价，建议使用 [ ]
- [[ EXPRESSION ]]

查看帮助

```bash
[root@centos8 data]# help [
[root@centos8 data]# help test
选项
	  -a FILE        True if file exists.
      -b FILE        True if file is block special.
      -c FILE        True if file is character special.
      -d FILE        True if file is a directory.
      -e FILE        True if file exists.
      -f FILE        True if file exists and is a regular file.
      -g FILE        True if file is set-group-id.
      -h FILE        True if file is a symbolic link.
      -L FILE        True if file is a symbolic link.
      -k FILE        True if file has its `sticky' bit set.
      -p FILE        True if file is a named pipe.
      -r FILE        True if file is readable by you.
      -s FILE        True if file exists and is not empty.
      -S FILE        True if file is a socket.
      -t FD          True if FD is opened on a terminal.
      -u FILE        True if the file is set-user-id.
      -w FILE        True if the file is writable by you.
      -x FILE        True if the file is executable by you.
      -O FILE        True if the file is effectively owned by you.
      -G FILE        True if the file is effectively owned by your group.
      -N FILE        True if the file has been modified since it was last read.
    
    String operators:
    
      -z ”STRING“      True if string is empty.   #字符串建议加上双引号
    
      -n STRING
         STRING      True if string is not empty.
    
      STRING1 = STRING2   #注意 = 前后有空格，没有的话那是
                     True if the strings are equal.
      STRING1 != STRING2
                     True if the strings are not equal.
      STRING1 < STRING2
                     True if STRING1 sorts before STRING2 lexicographically.
      STRING1 > STRING2
                     True if STRING1 sorts after STRING2 lexicographically.
   
    Other operators:
#变量测试    
      -o OPTION      True if the shell option OPTION is enabled.
      -v VAR         True if the shell variable VAR is set.
      -R VAR         True if the shell variable VAR is set and is a name
                     reference.
      ! EXPR         True if expr is false.
      EXPR1 -a EXPR2 True if both expr1 AND expr2 are true.
      EXPR1 -o EXPR2 True if either expr1 OR expr2 is true.
#数值测试    
      arg1 OP arg2   Arithmetic tests.  OP is one of -eq, -ne,
                     -lt, -le, -gt, or -ge.
    
    Arithmetic binary operators return true if ARG1 is equal, not-equal,
    less-than, less-than-or-equal, greater-than, or greater-than-or-equal
    than ARG2.
```

##### 1.9.1变量测试

范例

```bash
#判断 NAME 变量是否定义
[ -v NAME ] 
#判断 NAME 变量是否定义并且是否引用
[ -R NAME ]

[root@centos8 data]# echo $name
[root@centos8 data]# [ -v name ]
[root@centos8 data]# echo $?
1
[root@centos8 data]# name="xiaoming"
[root@centos8 data]# [ -v name ]
[root@centos8 data]# echo $?
0

[root@centos8 data]# [ -R name ]
[root@centos8 data]# echo $?
1
[root@centos8 data]# [ -v name ]
[root@centos8 data]# echo $?
0
```

##### 1.9.1数值测试

范例

```bash
[root@centos8 data]# i=10;j=20;
[root@centos8 data]# [ i -eq j ]
-bash: [: i: integer expression expected
[root@centos8 data]# [ $i -eq $j ]
[root@centos8 data]# echo $?
1
```

##### 1.9.2字符串测试

范例

```bash
[root@centos8 data]# echo $name

[root@centos8 data]# [ -z $name ]
[root@centos8 data]# echo $?
0
[root@centos8 data]# unset name
[root@centos8 data]# [ -z $name ]
[root@centos8 data]# echo $?
0
[root@centos8 data]# name=xiaoming
[root@centos8 data]# [ -z "$name" ]
[root@centos8 data]# echo $?
1
```

[ ]用法

```bash
[[]] 用法，建议，当使用正则表达式或通配符使用，一般情况使用 [ ]

== 左侧字符串是否和右侧的PATTERN相同
 注意:此表达式用于[[ ]]中，PATTERN为通配符

=~ 左侧字符串是否能够被右侧的正则表达式的PATTERN所匹配
 注意: 此表达式用于[[ ]]中；扩展的正则表达式
```

范例

```bash
[root@centos8 data]# NAME=linux
[root@centos8 data]# [[ "$NAME" == linu\* ]]
[root@centos8 data]# echo $?
1
[root@centos8 data]# [[ "$NAME" == linu* ]]
[root@centos8 data]# echo $?
0
#结论：[[ == ]] == 右侧的 * 做为通配符，不要加“”，只想做为*, 需要加“” 或转义
```

##### 1.9.3 文件测试

范例

```bash
[root@centos8 data]# [ -a /data/test ]
[root@centos8 data]# echo $?
0
[root@centos8 data]# ![ -a /data/test ]
[ -a /data/test ] -a /data/test ]
-bash: [: too many arguments
[root@centos8 data]# ! [ -a /data/test ]
[root@centos8 data]# echo $?
1
[root@centos8 data]# [ -w /etc/shadow ]
[root@centos8 data]# echo $?
0
```

### 1.10 () 和 {}

（CMD1;CMD2;...）和 { CMD1;CMD2;...; } 都可以将多个命令组合在一起，批量执行。使用man bash可以查看帮助

区别

( list ) 会开启子shell,并且list中变量赋值及内部命令执行后,将不再影响后续的环境

{ list; } 不会启子shell, 在当前shell中运行,会影响当前shell环境

范例

```bash
[root@centos8 data]# echo $BASHPID
164220
[root@centos8 data]# ( echo $BASHPID;sleep 100)
164450
sshd(976)───sshd(164206)───sshd(164219)───bash(164220)─┬─bash(164449)───sleep(164450)
[root@centos8 data]# echo $BASHPID
164220
[root@centos8 data]# { echo $BASHPID;}
164220
```

### 1.11 组合测试条件

##### 1.11.1 第一种方式

 [ EXPRESSION1 -a EXPRESSION2 ] 并且，EXPRESSION1和EXPRESSION2都是真，结果才为真

 [ EXPRESSION1 -o EXPRESSION2 ] 或者，EXPRESSION1和EXPRESSION2只要有一个真，结果就为 真

 [ ! EXPRESSION ] 取反

范例

```bash
[root@centos8 data]# FILE=/data/test
[root@centos8 data]# [ -f $FILE -a -x $FILE ]
[root@centos8 data]# echo $?
1
[root@centos8 data]# chmod +x /data/test
[root@centos8 data]# ll
total 20
-rw-r--r-- 1 root root   0 Jan  9 22:28 a.log
-rw-r--r-- 1 root root 181 Jan  4 18:29 arg.sh
-rw-r--r-- 1 root root   1 Jan  6 19:17 lx.txt
-rw-r--r-- 1 root root 116 Jan  5 15:50 rm.sh
-rw-r--r-- 1 root root 403 Jan  2 17:39 systeminfo.sh
-rwxr-xr-x 1 root root 110 Jan  3 10:17 test
[root@centos8 data]# [ -f $FILE -a -x $FILE ]
[root@centos8 data]# echo $?
0
```

##### 1.11.2 第二种方式

COMMAND1 && COMMAND2 并且，短路与，代表条件性的AND THEN 如果COMMAND1 成功,将执行COMMAND2,否则,将不执行COMMAND2

COMMAND1 || COMMAND2 或者，短路或，代表条件性的OR ELSE 如果COMMAND1 成功,将不执行COMMAND2,否则,将执行COMMAND2

! COMMAND #非,取反

范例

```bash
[root@centos8 data]# test "A" = "B" && echo "string are equal"
[root@centos8 data]# test "A" = "A" && echo "string are equal"
string are equal

[root@centos8 data]# ll
-rw-r--r-- 1 root root 110 Jan  3 10:17 test.sh
[root@centos8 data]# [ -f $FILE ] && [[ $FILE =~ \.sh$ ]] && chmod +x $FILE

[root@centos8 data]# ll
-rwxr-xr-x 1 root root 110 Jan  3 10:17 test.sh
```

&&与||组合使用

```bash
[root@centos8 data]# NAME=user;id $NAME && echo "$NAME is exist" || echo "$NAME is not exist"
uid=1001(user) gid=1001(user) groups=1001(user)
user is exist
#注意：如果&& 和 || 混合使用，&& 要在前，|| 放在后。否则会出现逻辑错误
```

### 1.12 read命令

read：使用read来把输入值分配给一个或多个shell变量，read从标准输入中读取值，给每个单词分配一个变 量，所有剩余单词都被分配给最后一个变量，如果变量名没有指定，默认标准输入的值赋值给系统内置 变量REPLY

```bash
read [options] [name ...]
```

常用选项

- -p 指定要显示的提示
- -s 静默输入，一般用于密码
- -n N 指定输入的字符长度N
- -d '字符' 输入结束符
- -t N TIMEOUT为N秒

范例

```bash
[root@centos8 data]# read
username
[root@centos8 data]# echo $REPLY
username
[root@centos8 data]# read NAME SEX
xiaoming boy
[root@centos8 data]# echo $NAME $SEX
xiaoming boy
[root@centos8 data]# read -p "please input your name:" NAME
please input your name:daxiong 
[root@centos8 data]# echo $NAME
daxiong
```

```bash
面试题 read和输入重定向
[root@centos8 data]# echo 1 2 | read x y;echo $x $y

[root@centos8 data]# echo 1 2 | (read x y;echo $x $y)
1 2
#Each command in a pipeline is executed as a separate process (i.e., in  a subshell)
#管道符把每个命令执行都在独立的子进程中，所以第一种情况父进程无法读取子进程的变量
```

## 2 bash的配置文件

### 2.1 配置文件分类

全局生效配置文件

```bash
/etc/profile
/etc/profile.d/*.sh
/etc/bashrc
```

个人用户生效配置文件

```bash
~/.bash_profile
~/.bashrc
```

### 2.2 配置文件加载顺序

##### 2.2.1 交互式登录顺序

- 直接通过终端输入账号密码登录
- 使用 su - UserName 切换的用户

配置文件加载顺序

```bash
/etc/profile.d/*.sh
/etc/bashrc
/etc/profile
/etc/bashrc    #此文件执行两次
.bashrc
.bash_profile
```

##### 2.2.2 非交互式顺序

- su UserName
- 图形界面下打开的终端
- 执行脚本
- 任何其他的bash实例

**配置文件加载顺序**

```bash
/etc/profile.d/*.sh
/etc/bashrc
.bashrc
```

### 2.3 配置文件分类

##### 2.3.1 Profile类

为交互式登录的shell提供配置

```bash
全局：/etc/profile, /etc/profile.d/*.sh
个人：~/.bash_profile
```

用途：

1）用于定义环境变量

2）运行脚本或命令

##### 2.3.2 Bashrc类

为非交互式和交互式登录的shell提供配置

```bash
全局：/etc/bashrc
个人：~/.bashrc
```

用途：

1）定义命令别名和函数

2）定义本地变量

##### 2.3.3 配置文件生效方式

1. 重新启动shell进程
2. source|. 配置文件

## 3.流程控制

### 3.1 条件判断if

格式

```bash
if: if COMMANDS; then COMMANDS; [ elif COMMANDS; then COMMANDS; ]... [ else COMMANDS; ] fi
```

范例

```bash
#BMI指数
[root@centos8 script]# cat bmi.sh 
#!/bin/bash
read -p "请输入身高（单位m）" HIGH
if [[ ! "$HIGH" =~ ^[0-2]\.?[0-9]{,2}$ ]];then
        echo "输入身高错误"
        exit 1
fi
read -p "请输入体重（单位kg）" WEIGHT
if [[ ! "$WEIGHT" =~ ^[0-9]{1,3}$ ]];then
        echo "输入体重错误"
        exit 1
fi
BMI=`echo $WEIGHT/$HIGH^2|bc`
if [ "$BMI" -le 18 ];then
        echo "你太瘦了多吃点"
elif [ "$BMI" -lt 24 ];then
        echo "你的身材很棒"
else
        echo "你太胖了，加强运动"
fi
```

### 3.2 条件判断 case 语句

格式

```bash
case: case WORD in [PATTERN [| PATTERN]...) COMMANDS ;;]... esac
    Execute commands based on pattern matching.	
    
#case支持全局通配符
*: 任意长度任意字符
?: 任意单个字符
[]：指定范围内的任意单个字符
|:   或，如 a或b
```

范例

```bash
#运维表
#!/bin/bash
echo -en "\E[$[RANDOM%7+31];1m"
cat <<EOF
1)备份数据库
2）清理日志
3）软件升级
4）软件回退
5）删库跑路
EOF
echo -en '\E[0m'
read -p "请输入上面的数字：" MENU
if [[ ! "$MENU" =~ ^[1-5]$ ]];then
        echo "你的输入有误"
        exit 1
fi
case $MENU in
1)
    echo "备份成功";;
2)
    echo "清理日志";;                  
3)
    echo "清理日志";;        
4)
    echo "清理日志";;        
5)          
    echo "清理日志";;                  
esac
```

### 3.3 循环

##### 3.3.2 for循环

格式：

```bash
#格式1
for NAME [in WORDS ... ] ; do COMMANDS; done
#格式2
for (( 控制变量初始化;条件判断表达式;控制变量的修正表达式 )); do COMMANDS; done
```

循环列表生成方式：

- 直接给出列表

- 整数列表

  {start..end}

  $(seq [start [step]] end)

- 返回列表的命令

  $(COMMAND)

- 使用*.sh

- 变量引用，如：$@,$*

##### 3.3.2 while循环

格式：

```bash
while COMMANDS; do COMMANDS; done

while CONDITION; do
 	循环体
done
#进入条件：CONDITION为true
#当循环次数不确定时建议使用while
```

![image-20230814172608582](C:\Users\user\Desktop\Linux Memo\image-20230814172608582.png)

##### 3.3.3 until 循环

格式：

```bash
until COMMANDS; do COMMANDS; done
until CONDITION; do
 		循环体
done

#进入条件： CONDITION 为false
#退出条件： CONDITION 为true
```

3.3.4 循环控制语句 continue

continue [N]：提前结束第N层的本轮循环，而直接进入下一轮判断；最内层为第1层

范例 ：

```bash
[root@centos8 script]$vim continue.sh
for ((i=0;i<=10;i++));do
    for ((j=0;j<10;j++));do
        [ $j -eq 5 ] && continue 2            
        echo -e $j'\c'
    done
    echo ----------------------
done
[root@centos8 script]$bash continue.sh 
0123401234012340123401234012340123401234012340123401234 
```

##### 3.3.5 循环控制语句 break

break [N]：提前结束第N层整个循环，最内层为第1层

范例：



```bash
[root@centos8 script]$vim continue.sh
for ((i=0;i<=10;i++));do
    for ((j=0;j<10;j++));do
        [ $j -eq 6 ] && break 2                       
        echo -e $j'\c'
    done
    echo ----------------------
done
[root@centos8 script]$bash continue.sh 
012345
```

##### 3.3.6 循环控制 shift 命令

shift [n] 用于将参量列表 list 左移指定次数，缺省为左移一次。

范例：

```bash
[root@centos8 script]$cat doit.sh 
#!/bin/bash
until [ -z "$1" ]
do 
    echo "$1"
    shift
done
echo
[root@centos8 script]$bash doit.sh 1 2 3 4 5
1
2
3
4
5
```

##### 3.3.7 select 循环与菜单

格式：

```bash
select NAME [in WORDS ... ;] do COMMANDS; done

select variable in list ;do
 			循环体命令
done
```

- select 循环主要用于创建菜单，按数字顺序排列的菜单项显示在标准错误上，并显示 PS3 提示符，等待用户输入
- 用户输入被保存在内置变量 REPLY 中
- select 是个无限循环，因此要用 break 命令退出循环，或用 exit 命令终止脚本。也可以按 ctrl+c 退出循环

范例：

```bash
[root@centos8 script]$cat select.sh
#!/bin/bash
sum=0
PS3="请输入1-6"
select menu in 北京烤鸭 佛跳墙 小龙虾 羊蝎子 火锅 点菜结束 ;do
    case $REPLY in 
    1)
        echo $menu 价格是100
        let sum+=100
        ;;
    2)
        echo $menu 价格是88
        let sum+=88
        ;;
    3)
        echo $menu 价格是10
        let sum+=10
        ;;
    4)
        echo $menu 价格是190
        let sum+=190
        ;;
    5)
        echo $menu 价格是17
        let sum+=17
        ;;
    6)
        break;;
        esac
   done
   echo "$sum"
```

## 4 函数

Shell程序在子Shell中运行，而Shell函数在当前Shell中运行。因此在当前Shell中，函数可对shell中变量 进行修改

### 4.1 函数管理

##### 4.1.1 定义函数

```bash
#语法一:
function name { 
	COMMANDS ; 
} 
#语法二：
function name () { 
	COMMANDS ; 
}
#语法三：
func_name （）{
 	COMMANDS ;
}
```

##### 4.1.2 查看函数

```bash
#显示已定义函数名称及定义
delcare -f
#显示已定义函数名称
delcare -F
```

4.1.3 删除函数

```bash
unset func_name
```

### 4.2 函数调用

##### 4.2.1 交互式环境函数调用

范例：

```bash
[root@centos8 script]$dir() { ls -l;}
[root@centos8 script]$dir
total 44
-rw-r--r-- 1 root root 482 Mar  4 11:39 continue.sh

[root@centos8 script]$declare -f dir
dir () 
{ 
    ls --color=auto -l
}
```

##### 4.2.2 脚本中定义及使用

注意：函数使用前需先定义

```bash
[root@centos8 script]$bash function.sh 
hello
[root@centos8 script]$cat function.sh 
#!/bin/bash
function hello(){
    echo "hello"
}
hello
```

##### 4.2.3 调用函数文件

格式：

```bash
. filename	或 source filename	
```

范例：

```bash
[root@centos8 script]$. function
[root@centos8 script]$hello
hello
[root@centos8 script]$cat function 
#!/bin/bash
function hello(){
    echo "hello"
}
```

### 4.3 环境函数

定义：

```bash
export -f function_name
declare -xf function_name
```

查看：

```bash
export -f
declare -xf
```

### 4.4 函数参数

- 传递参数给函数：在函数名后面以空白分隔给定参数列表即可
- 在函数体中当中，可使用$1, $2, ...调用这些参数；还可以使用$@, $*, $#等特殊变量,函数名称存储于$FUNCNAME变量中

### 4.5 函数中变量有效范围

- 普通变量：只在当前shell进程有效，为执行脚本会启动专用子shell进程；因此，本地变量的作用 范围是当前shell脚本程序文件，包括脚本中的函数
- 环境变量：当前shell和子shell有效
- 本地变量：函数的生命周期；函数结束时变量被自动销毁

由于普通变量和局部变量会冲突，建议在函数中只使用本地变量

```bash
[root@centos8 script]$name=hello
[root@centos8 script]$echo $name
hello
[root@centos8 script]$. function 
[root@centos8 script]$hello
[root@centos8 script]$echo $name
tom
[root@centos8 script]$cat function 
#!/bin/bash
function hello(){
   name=tom
}
```

在函数中定义本地变量的方法

```bash
local NAME=VALUE
```

## 5 其他脚本工具

### 5.1 trap

捕捉系统信号并且执行相应动作

- trap '触发指令' 信号

  进程收到系统发出的指定信号后，将执行自定义指令，而不会执行原操作

- trap '' 信号

  忽略信号的操作

- trap '-' 信号

  恢复原信号的操作

- trap -p

  列出自定义信号操作

- trap finish EXIT

  当脚本退出时，执行finish函数

- signal_spec可以是简写，信号数字、信号全称（int、2、SIGINT）

范例：

```bash
#捕捉到ctrl+c时什么也不执行，使用kill -L查看信号
[root@centos8 script]$cat trap.sh 
#!/bin/bash
trap '' int quit
for((i=0;i<=10;i++))
do
       sleep 1
       echo $i
done
```

### 5.2 mktemp

创建并显示临时文件或目录，主要用于避免文件命名冲突

格式：

```bash
mktemp [OPTION]... [TEMPLATE]
```

注意：TEMPLATE: filenameXXX，X至少要出现三个而且必须是大写

常见选项：

- -d 创建目录
- -p 指定创建临时文件的目录

范例：

```bash
[root@centos8 test]$mktemp testXXX
testDji

[root@centos8 test]$mktemp --tmpdir= testXXX
/tmp/testle5

#rm -rf命令的实现
[root@centos8 script]$cat rm_rf.sh 
#!/bin/bash
DIR=`mktemp -d /tmp/$(date +%F_%H-%M-%S)XXX`
mv $* $DIR
echo $* is move to $DIR
```

### 5.3 install

install 功能相当于cp，chmod，chown，chgrp 等相关工具的集合

格式：



```bash
install [OPTION]... [-T] SOURCE DEST 单文件
install [OPTION]... SOURCE... DIRECTORY
install [OPTION]... -t DIRECTORY SOURCE...
install [OPTION]... -d DIRECTORY...创建空目录
```

选项：

- -m MODE.默认755
- -o OWNER
- -g GROUP
- -d DIRNAME 目录

### 5.4 expect

用于实现处理交互命令，实现自动完成交互

格式：



```bash
expect [选项] [ -c cmds ] [ [ -[f|b] ] cmdfile ] [ args ]
```

选项：

- -c：从命令执行expect脚本
- -d：输出调试信息

expect中相关命令：

- spawn 启动新的进程
- expect 从进程接收字符串
- send 用于向进程发送字符串
- interact 允许用户交互
- exp_continue 匹配多个字符串在执行动作后加此命令

```bash
[root@centos8 script]$expect  -c 'expect "\n" {send "pressed enter\n"}'

pressed enter
[root@centos8 script]$expect  -dc 'expect "\n" {send "pressed enter\n"}'
expect version 5.45.4
expect: does "" (spawn_id exp0) match glob pattern "\n"? no

expect: does "\n" (spawn_id exp0) match glob pattern "\n"? yes
expect: set expect_out(0,string) "\n"
expect: set expect_out(spawn_id) "exp0"
expect: set expect_out(buffer) "\n"
send: sending "pressed enter\n" to { exp0 pressed enter
}
argv[0] = expect  argv[1] = -dc  argv[2] = expect "\n" {send "pressed enter\n"}  
set argc 0
set argv0 "expect"
set argv ""
```

单分支语法：

```bash
passwdexpect1.5> expect "hi" {send "hello\n"}
hi
hello
```

多分支语法：

```bash
#语法1
[root@centos8 test]#expect
expect1.1> expect "hi" { send "You said hi\n" } "hehe" { send "Hehe yourself\n"
} "bye" { send "Good bye\n" }
hehe
Hehe yourself

#语法2
expect {
 "hi" { send "You said hi\n"}
 "hehe" { send "Hehe yourself\n"}
 "bye" { send " Good bye\n"}
}
```

定义变量：

```bash
#set	变量名	值
set ip 10.0.0.7
```

位置参数：

```bash
#!/usr/bin/expect
set ip [lindex $argv 0] 
```

## 6 数组

### 6.1 声明数组

```bash
#普通数组可以不事先声明,直接使用
declare -a ARRAY_NAME
#关联数组必须先声明,再使用：关联数组就是自定义索引格式的数组
declare -A ARRAY_NAME
```

### 6.2 数组赋值

1）一次赋值一个元素

```bash
ARRAY_NAME[INDEX]=VALUE
```

2）一次赋值多个元素

```bash
ARRAY_NAME=(val1 val2 val3)
```

范例：

```bash
name=("tom" "xiaoming" "xiaohong")
num=({1..10})
file=(*.sh)
```

3）交互式赋值

```bash
read -a ARRAY		
```

### 6.3 查看所有数组

显示所有数组

```bash
declear -a
```

### 6.4 引用数组

引用数组元素

```bash
${ARRAY-NAME[INDEX]}
```

范例：

```bash
[root@centos8 script]$declare -a title
[root@centos8 script]$title=({1..3})
[root@centos8 script]$echo ${title}
1
```

引用数组所有元素

```bash
${ARRAY_NAME[*]}
${ARRAY_NAME[@]}
```

数组中元素的个数

```bash
${#ARRAY_NAME[*]}
${#ARRAY_NAME[@]}
```

### 6.5 删除数组

删除数组中的某元素

```bash
unset ARRAY[INDEX]
```

删除整个数组

```bash
unset ARRAY
```

练习：

1.分别使用shell和expect实现远程登陆主机

```bash
#expect语法实现(允许用户交互，不会自动退出)
[root@centos8 script]# cat expect1.sh 
#!/usr/bin/expect
set ip [lindex $argv 0]
set user [lindex $argv 1]
set passwd [lindex $argv 2]
spawn ssh $user@$ip
expect {
   "yes/no" {send "yes\n";exp_continue}
   "password" {send "$passwd\n"}

}
interact

#shell语法实现（此方式会在到达timeout时间后自动退出，默认timeout时间为10s）
[root@centos8 script]# cat expect.sh 
#/bin/bash
ip=10.0.0.203
user=root
passwd=123321
expect << EOF
set timeout 2
spawn ssh $user@$ip
expect {
	"yes/no" {send "yes\n";exp_continue}
	"password" {send "$passwd\n"}
}
expect eof
EOF
```

2.生成10个随机数保存于数组中，并找出其中最大值和最小值

```bash
[root@centos8 script]$cat random.sh 
#!/bin/bash
declare -i max min
declare -a random
for((i=0;i<10;i++));do
    random[$i]=$RANDOM
    if [ $i -eq 0 ];then
        min=${random[0]} 
        max=${random[0]}
    else
        [ ${random[$i]} -gt $max ] && max=${random[$i]}
        [ ${random[$i]} -lt $min ] && min=${random[$i]}
    fi
done
echo "all num are ${random[*]}"
echo "max num is $max"
echo "min num is $min"
```

3.输入若干个数存入数组中，采用冒泡算法进行排序

```bash
[root@centos8 script]$cat sort.sh 
#!/bin/bash
declare -a num
num=(1 10 6 5 8 7 9 4 3 2)
echo 调换顺序之前数组顺序 ${num[*]}
for((j=9;j>0;j--));do
    for ((i=0;i<j;i++));do
       if [ ${num[$i]} -gt ${num[$[i+1]]} ];then
         b=${num[$i]} 
        num[$i]=${num[$[i+1]]}
        num[$[i+1]]=$b
       fi
    done
done
echo 调换顺序之后数组顺序 ${num[*]}
```

# 7.文件查找和软件解压缩 软件包管理

## 1.文件查找

### 1.1 locate

locate：通过名字查找文件（模糊查找）

locate 查询系统上预建的文件索引数据库 /var/lib/mlocate/mlocate.db

若数据库不存在可以执行updatedb可以更新数据库构建索引

索引构建过程需要遍历整个根文件系统，很消耗资源，避免在高峰时间段使用

格式：

```bash
locate [OPTION]... PATTERN...
```

常用选项

- -i 不区分大小写
- -n N 只列举前N个匹配项目
- -r 使用基本正则表达式

范例：

```bash
#使用正则来搜索以.newfile结尾的文件
[root@centos8 data]# touch test.newfile
[root@centos8 data]# locate -r '\.newfile$'
[root@centos8 data]# updatedb
[root@centos8 data]# locate -r '\.newfile$'
/data/test.newfile

#查找以.conf结尾的前10个文件
[root@centos8 data]# locate -n 10 -r '\.conf$'
/boot/loader/entries/20200914151302543507749550121287-0-rescue.conf
/boot/loader/entries/20200914151302543507749550121287-4.18.0-193.14.2.el8_2.x86_64.conf
/boot/loader/entries/20200914151302543507749550121287-4.18.0-193.el8.x86_64.conf
/etc/chrony.conf
/etc/dracut.conf
/etc/fprintd.conf
/etc/host.conf
/etc/idmapd.conf
/etc/kdump.conf
/etc/krb5.conf
```

### 1.2 find

find：在目录层次结构中搜索文件（精确查找）

find的工作机制使得它可以完成实时查找，而且拥有很多查询条件，实现各种方式查找

格式：

```bash
find [OPTION]... [查找路径] [查找条件] [处理动作]
```

##### 1.2.1 指定搜索目录层级

-maxdepth level 最大搜索目录深度,指定目录下的文件为第1级

-mindepth level 最小搜索目录深度

范例：

```bash
[root@centos8 data]# find /data -maxdepth 1
```

##### 1.2.2 根据文件名和inode查找

- -name "文件名称"：支持使用glob，如：*, ?, [], [^],通配符要加双引号引起来

- -iname "文件名称"：不区分字母大小写

- -inum n 按inode号查找

  ![image-20230816112041312](C:\Users\user\Desktop\Linux Memo\image-20230816112041312.png)

- -samefile name 相同inode号的文件

- -links n 链接数为n的文件

- -regex “PATTERN”：以PATTERN匹配整个文件路径，而非文件名称

范例：

```bash
[root@centos8 data]# find /data -name 1.txt
/data/script/level2/1.txt
/data/1.txt

[root@centos8 data]# find /data -name "*.txt"
/data/lx.txt
/data/script/level2/1.txt
/data/1.txt
/data/2.txt
/data/3.txt

[root@centos8 data]# ll -i
   830873 -rw-r--r-- 2 root root   0 Jan 14 17:23 1.txt
   830875 -rw-r--r-- 1 root root   0 Jan 14 17:23 3.txt
   830873 -rw-r--r-- 2 root root   0 Jan 14 17:23 9.txt
[root@centos8 data]# find /data/ -samefile 1.txt 
/data/1.txt
/data/9.txt
```

##### 1.2.3 根据属主、属组查找

- -user USERNAME：查找属主为指定用户(UID)的文件
- -group GRPNAME: 查找属组为指定组(GID)的文件
- -uid UserID：查找属主为指定的UID号的文件
- -gid GroupID：查找属组为指定的GID号的文件
- -nouser：查找没有属主的文件
- -nogroup：查找没有属组的文件

##### 1.2.4 根据文件类型查找

- -type TYPE
  TYPE可以是以下形式：

   f: 普通文件
  d: 目录文件
  l: 符号链接文件
  s：套接字文件
  b: 块设备文件
  c: 字符设备文件
  p: 管道文件

范例：

```bash
[root@centos8 data]# find /data/ -type d 
/data/
/data/script
/data/script/level2
/data/test
```

##### 1.2.5 空文件或目录

- -empty

范例：

```bash
[root@centos8 data]# find /data -type d -empty 
/data/test
/data/empty
```

##### 1.2.6 组合条件查询

- 与：-a ，默认多个条件是与关系
- 或：-o
- 非：-not !

小技巧

 !A -a !B = !(A -o B)

 !A -o !B = !(A -a B)

范例：

```bash
[root@centos8 data]# find /data -type d -empty 
/data/test
/data/empty
[root@centos8 data]# find /data -type d -a -empty 
/data/test
/data/empty

[root@centos8 data]# find /data ! \( -type f -o -empty \)
/data
/data/script
/data/script/level2
```

##### 1.2.7 排除目录

- -path pattern 只查找pattern目录，不展示工作树
- -prune 配合-path选项排除目录

格式

```
find [查找目录] -path '排除目录' -prune -o [查找条件] -print
```

范例：

```bash
[root@centos8 data]# find /data -path '/data/script'
/data/script
[root@centos8 data]# find /data -path '/data/script' -prune -o -print
/data
/data/systeminfo.sh
/data/arg.sh

#查找/data下，除test和script目录的其它所有目录
[root@Zabbix-MySql data]# ll
总用量 0
-rw-r--r-- 1 root root 0 1月  16 11:31 1.txt
-rw-r--r-- 1 root root 0 1月  16 11:31 2.txt
-rw-r--r-- 1 root root 0 1月  16 11:31 3.txt
drwxr-xr-x 2 root root 6 1月  16 11:30 empty
drwxr-xr-x 2 root root 6 1月  16 11:30 script
drwxr-xr-x 2 root root 6 1月  16 11:30 test
[root@Zabbix-MySql data]# find /data \( -path '/data/test' -o -path '/data/script' \) -a -prune -o -type d -print
/data
/data/empty
```

##### 1.2.8 根据文件大小来查找

-size [+|-]#UNIT

\#UNIT: (#-1, #]

 如：6k 表示(5k,6k]

-#UNIT：[0,#-1]

 如：-6k 表示[0,5k]

+#UNIT：(#,∞)

 如：+6k 表示(6k,∞)

范例：

```bash
find /  -size +10G
```

##### 1.2.9 根据时间戳

以“天”为单位

- -atime [+|-]#

  #: [#,#+1)

  +#: [#+1,∞]

  -#: [0,#)

- -mtime [+|-]#

- -ctime [+|-]#

以“分钟”为单位

- -amin [+|-]#
- -mmin [+|-]#
- -cmin [+|-]#

##### 1.2.10 根据权限查找

- -perm [/ | -]MODE

  MODE：精确权限匹配

  /MODE：任何一类(u,g,o)对象的权限中只要能一位匹配即可，或关系

  -MODE：每一类对象都必须同时拥有指定权限，与关系

  0 表示不关注

范例：

```bash
[root@centos8 data]# find /data/ -perm -7000 -ls
   830873      0 ---S--S--T   1  root     root            0 Jan 14 17:23 /data/1.txt
[root@centos8 data]# ll 1.txt 2.txt
---S--S--T 1 root root 0 Jan 14 17:23 1.txt
---S--S--- 1 root root 0 Jan 14 17:23 2.txt
[root@centos8 data]# find /data/ -perm /7000 -ls
   830873      0 ---S--S--T   1  root     root            0 Jan 14 17:23 /data/1.txt
   830874      0 ---S--S---   1  root     root            0 Jan 14 17:23 /data/2.txt
```

##### 1.2.11 处理动作

- -print：默认的处理动作，显示至屏幕
- -ls：类似于对查找到的文件执行“ls -l”命令
- -fls file：查找到的所有文件的长格式信息保存至指定文件中，相当于 -ls > file
- -delete：删除查找到的文件，慎用！
- -ok COMMAND {} ; 对查找到的每个文件执行由COMMAND指定的命令，对于每个文件执行命令之前，都会交互式要求用户确认
- -exec COMMAND {} ; 对查找到的每个文件执行由COMMAND指定的命令
- {}: 用于引用查找到的文件名称自身

范例：

```bash
#找到data目录.conf结尾文件，并备份为.bak结尾文件
[root@Zabbix-MySql data]#find /data -name "*.conf" -exec cp {} {}.bak \;
[root@Zabbix-MySql data]#ll
-rw-r--r-- 1 root root 0 1月  16 15:26 lx.conf
-rw-r--r-- 1 root root 0 1月  16 16:00 lx.conf.bak
```

练习：

```bash
1.查找/etc目录下大于1M且类型未普通文件的所有文件
[root@centos8 data]# find /etc -size +1M -type f
/etc/selinux/targeted/policy/policy.31
/etc/udev/hwdb.bin
2.查找当前系统上没有属主或属组，且最近一个周内曾被访问过的文件或目录
[root@centos8 data]# find / -nouser -o -nogroup -a -atime -7 \( -type f -o -type d \)
/var/spool/mail/mandriva
/home/mandriva
/home/mandriva/.bash_logout
/home/mandriva/.bash_profile
/home/mandriva/.bashrc

3.查找/etc目录下至少有一类用户没有执行权限的文件
[root@centos8 data]# find /etc/ ! -perm -111 -a -type f -ls
```

## 2 压缩和解压缩

### 2.1 compress和uncompress

compress：由于压缩比比其他软件底，不常用

格式

```bash
compress Options [file ...]
uncompress file.Z #解压缩
```

常用选项：

- -d 解压缩，相当于uncompress
- -c 结果输出至标准输出,不删除原文件
- -v 显示解压缩过程

### 2.2 gzip和gunzip

格式：

```bash
gzip [OPTION]... FILE ...
```

常用选项：

- -k keep, 保留原文件,CentOS 8 新特性
- -d 解压缩，相当于gunzip
- -c 结果输出至标准输出，保留原文件不改变
- -# 指定压缩比，#取值为1-9，值越大压缩比越大

范例：

```bash
[root@centos8 data]# gzip -k message 
[root@centos8 data]# ll
-rw------- 1 root root  15366 Jan 16 18:00 message.gz

#不解压查看压缩文件内容
zcat 作用等同于 gunzip -c

#高级用法
#压缩多个文件
cat file1 file2 | gzip > foo.gz
gzip -c file1 file2 > foo.gz
```

### 2.3 bzip2和bunzip2

格式：

```bash
bzip2 [OPTION]... FILE ...
```

常用选项：

- -k keep, 保留原文件
- -d 解压缩
- -c 结果输出至标准输出，保留原文件不改变
- -# 1-9，压缩比，默认为9

范例：

```bash
[root@centos8 data]# bzip2 -k message
[root@centos8 data]# ll
-rw------- 1 root root   8955 Jan 16 18:00 message.bz2

bzcat file.bz2		不解压缩的前提下查看文本文件内容
```

### 2.4 xz和unxz

格式：

```bash
xz [OPTION]... FILE ...
```

常用选项：

- -k keep, 保留原文件
- -d 解压缩
- -c 结果输出至标准输出，保留原文件不改变
- -# 压缩比，取值1-9，默认为6

范例：

```bash
[root@centos8 data]# xz message
[root@centos8 data]# ll
-rw------- 1 root root   8060 Jan 16 18:00 message.xz

unxz file.xz 解压缩
xzcat file.xz 不显式解压缩的前提下查看文本文件内容
```

### 2.5 zip和unzip

zip 不同之处在于可以实现打包目录，不加选项实现多个文件压缩，但可能回丢失文件属性信息，如：属主和属组等

格式

```bash
zip [OPTION] [zipfile [file ...]]
```

常见选项：

- -r 打包整个目录内容
- -d 指定路径

范例：

```bash
[root@centos8 home]# zip -r /data/home.zip /home/
[root@centos8 data]# ll
-rw-r--r-- 1 root root 3302361 Jan 16 18:41 home.zip

#解压缩至指定目录,如果指定目录不存在，会在其父目录（必须事先存在）下自动生成
unzip /backup/sysconfig.zip  -d /tmp/config 

#-p 表示管道
unzip -p message.zip   > message 
```

## 3 打包和解包

### 3.1 tar

可以对目录和多个文件打包一个文件，并且可以压缩，保留文件属性不丢失

格式：

```bash
tar [OPTION]... FILE ...
```

常见选项：

- -c 打包
- -f 指定文件名
- -v 显示打包过程
- -x 解包
- -t 预览
- -C 指定解包目录
- --exclude 排除文件
- -r 追加文件至归档
- -T 指定要打包的文件
- -X 指定排除在打包外的文件

范例：

```bash
[root@centos8 data]# tar -cf message1.tar message1 home.zip

[root@centos8 data]# tar -tf message1.tar
message1
home.zip

[root@centos8 data]# tar -xf message1.tar
[root@centos8 data]# ll
drwxr-xr-x 5 root root      50 Jan 16 18:59 home
-rw------- 1 root root   10240 Jan 16 20:05 message1

#结合压缩工具实现：归档并压缩 
-z 相当于gzip压缩工具
-j 相当于bzip2压缩工具
-J 相当于xz压缩工具
[root@centos8 data]# tar -zcf message.tar.gz message1 home.zip
[root@centos8 data]# ll
-rw-r--r-- 1 root root 3226623 Jan 16 20:13 message.tar.gz

#利用 tar 进行文件复制（速度比cp命令稍快）
[root@centos8 ~]#tar c /data/ | tar x -C /backup

tar zcvf mybackup.tgz -T /root/includefilelist -X /root/excludefilelist

tar zcvf /root/a.tgz --exclude=/app/host1 --exclude=/app/host2 /app
```

### 3.2 split

分割一个文件为多个文件

格式：

```bash
split [OPTION]... [FILE [PREFIX]]
```

常见选项：

- -b 指定分割文件的大小
- -d 使用从0开始的数字后缀，而不是字母

范例：

```bash
[root@centos8 data]# split -b 1M message1.tar /data/home/message
[root@centos8 data]# ll home
-rw-r--r-- 1 root root 1048576 Jan 16 23:33 messageaa
-rw-r--r-- 1 root root 1048576 Jan 16 23:33 messageab
-rw-r--r-- 1 root root 1048576 Jan 16 23:33 messageac
-rw-r--r-- 1 root root  172032 Jan 16 23:33 messagead

#将多个文件合并成一个文件
[root@centos8 data]# cat ./home/messagea* > message.tar
[root@centos8 data]# ll
-rw-r--r-- 1 root root 3317760 Jan 16 23:36 message.tar
```

练习：

```bash
1.打包/etc目录下所有conf结尾的文件，压缩包名称未当天的时间，并拷贝到/usr/local/src目录备份
[root@centos8 data]# tar -zcvf /usr/local/src/`date +%F`.tar.gz /etc/*.conf
[root@centos8 data]# ll /usr/local/src/
-rw-r--r-- 1 root root 37802 Jan 19 16:43 2021-01-19.tar.gz
```

## 4.软件包和包管理器

### 4.1 软件包介绍

开源软件最初只提供了.tar.gz的打包的源码文件，用户必须自已编译每个想在GNU/Linux上运行的软件。为了更加便利的方法来管理这些软件，出现了包管理系统。

##### 4.1.1 程序包管理器

软件包管理器功能：

将编译好的应用程序的各组成文件打包一个或几个程序包文件，利用包管理器可以方便快捷地实现程序 包的安装、卸载、查询、升级和校验等管理操作

主流的程序包管理器

- redhat：rpm文件, rpm 包管理器
- debian：deb文件, dpkg 包管理器

##### 4.1.2 包命名

源代码打包文件命名：

- name-VERSION.tar.gz|bz2|xz
- VERSION: major.minor.release

rpm包命名方式：

- name-VERSION-release.arch.rpm
- VERSION: major.minor.release

常见的arch：

- x86: i386, i486, i586, i686
- x86_64: x64, x86_64, amd64
- 跟平台无关：noarch

##### 4.1.3 程序包管理器相关文件

1）包文件组成 (每个包独有)

- 包内的文件
- 元数据，如：包的名称，版本，依赖性，描述等
- 可能会有包安装或卸载时运行的脚本

2）数据库(公共)：/var/lib/rpm

- 程序包名称及版本
- 依赖关系
- 功能说明
- 包安装后生成的各文件路径及校验码信息

##### 4.1.4 获取程序包的途径

1）系统发版的光盘或官方网站

CentOS镜像：

https://www.centos.org/download/

[http://mirrors.aliyun.com](http://mirrors.aliyun.com/)

[http://mirrors.sohu.com](http://mirrors.sohu.com/)

[http://mirrors.163.com](http://mirrors.163.com/)

Ubuntu 镜像：

http://cdimage.ubuntu.com/releases/

[http://releases.ubuntu.com](http://releases.ubuntu.com/)

2）第三方组织提供

https://fedoraproject.org/wiki/EPEL

https://mirrors.aliyun.com/epel/?spm=a2c6h.13651104.0.0.3bc47dfaZpesAr

[http://www.elrepo.org](http://www.elrepo.org/)

3）软件项目官方站点

http://yum.mariadb.org/10.4/centos8-amd64/rpms/

http://repo.mysql.com/yum/mysql-8.0-community/el/8/x86_64/

4）搜索引擎

[http://pkgs.org](http://pkgs.org/)

[http://rpmfind.net](http://rpmfind.net/)

[http://rpm.pbone.net](http://rpm.pbone.net/)

https://sourceforge.net/

5）自己制作

将源码文件，利用工具，如：rpmbuild，fpm等工具制作成rpm包文件

## 5.包管理器rpm

### 5.1 安装

格式：

```bash
rpm {-i|--install} [install-options] PACKAGE_FILE ...
```

选项：

- -v 打印详细信息
- -h 显示安装进度

安装选项【install-options】

- --test: 测试安装，但不真正执行安装，即dry run模式
- --nodeps：忽略依赖关系
- --nosignature: 不检查来源合法性
- --nodigest：不检查包完整性
- --noscripts：不执行程序包脚本

注意：由于rpm命令安装包大部分会产生包依赖问题，所以一般不使用rpm安装包

### 5.2 升级和降级

格式：

```bash
升级
rpm {-U|--upgrade} [install-options] PACKAGE_FILE...
rpm {-F|--freshen} [install-options] PACKAGE_FILE...
#upgrade：安装有旧版程序包，则“升级”，如果不存在旧版程序包，则“安装”
#freshen：安装有旧版程序包，则“升级”， 如果不存在旧版程序包，则不执行升级操作

降级
rpm {--oldpackage} [install-options] PACKAGE_FILE...
```

注意：1.不要对内核做升级操作；Linux支持多内核版本并存，因此直接安装新版本内核

 2.如果原程序包的配置文件安装后曾被修改，升级时，新版本提供的同一个配置文件不会直接覆盖老 版本 的配置文件，而把新版本文件重命名(FILENAME.rpmnew)后保留

### 5.3 包查询

格式：

```bash
rpm {-q|--query} [select-options] [query-options]
```

【select-options】

- -a：所有包
- -f：查看指定的文件由哪个程序包安装生成
- -p rpmfile：针对尚未安装的程序包文件做查询操作

【query-options】

- --changelog：查询rpm包的changelog
- -c：查询程序的配置文件
- -d：查询程序的文档
- -i：information
- -l：查看指定的程序包安装后生成的所有文件
- --scripts：程序包自带的脚本

和CAPABILITY相关

- --whatprovides CAPABILITY：查询指定的CAPABILITY由哪个包所提供
- --whatrequires CAPABILITY：查询指定的CAPABILITY被哪个包所依赖
- --provides：列出指定程序包所提供的CAPABILITY
- -R：查询指定的程序包所依赖的CAPABILITY

范例：

```bash
[root@centos8 data]# rpm -qf /etc/skel/.bashrc 
bash-4.4.19-10.el8.x86_64

[root@centos8 data]# rpm -ql tree
/usr/bin/tree
/usr/lib/.build-id
/usr/lib/.build-id/d8
/usr/lib/.build-id/d8/6d516d7cb07fb9334cb268af808119e33a5ac5
/usr/share/doc/tree
/usr/share/doc/tree/LICENSE
/usr/share/doc/tree/README
/usr/share/man/man1/tree.1.gz

[root@centos8 data]# rpm -qd tree
/usr/share/doc/tree/LICENSE
/usr/share/doc/tree/README
/usr/share/man/man1/tree.1.gz

[root@centos8 data]# rpm -q --scripts bash
postinstall scriptlet (using <lua>):
nl        = '\n'
sh        = '/bin/sh'..nl
...

[root@centos8 data]# rpm -q --whatrequires tree
no package requires tree
```

### 5.4 包卸载

格式：

```bash
rpm {-e|--erase} [--allmatches] [--nodeps] [--noscripts] [--notriggers] [--test]
PACKAGE_NAME ...
```

注意：当包卸载时，对应的配置文件不会删除， 以FILENAME.rpmsave形式保留

范例：强行删除rpm包，并恢复

```bash
[root@centos7 ~]#rpm -e rpm --nodeps
#重启进入rescue模式
#mkdir /mnt/cdrom
#mount /dev/sr0 /mnt/cdrom
#rpm -ivh /mnt/cdrom/Packages/rpm-4.11.3-40.el7.x86_64.rpm --root=/mnt/sysimage
#reboot
```

### 5.5 包校验

在安装包时，系统也会检查包的来源是否是合法的

自己校验步骤：

```bash
1.导入所需要公钥
rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
#查询密钥是否导入成功
rpm -qa “gpg-pubkey*”

2.检查包的完整性和签名
rpm -K|--checksig rpmfile  
```

范例：

```bash
[root@centos8 ~]#rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
[root@centos8 rpm-gpg]#rpm -K /misc/cd/AppStream/Packages/httpd-2.4.37-
16.module_el8.1.0+256+ae790463.x86_64.rpm
/misc/cd/AppStream/Packages/httpd-2.4.37-
16.module_el8.1.0+256+ae790463.x86_64.rpm: digests signatures OK

[root@centos8 ~]#rpm -qa "gpg-pubkey*"
gpg-pubkey-8483c65d-5ccc5b19
```

软件在安装时，会将包里的每个文件的元数据，如：大小，权限，所有者，时间等记录至rpm相关的数 据库中，可以用来检查包中的文件是否和当初安装时有所变化

格式：

```bash
rpm {-V|--verify} [select-options] [verify-options]
```

范例：

```bash
[root@centos8 data]# vim /etc/issue
[root@centos8 data]# cat /etc/issue
111
\S
Kernel \r on an \m
[root@centos8 data]# rpm -V centos-release 
S.5....T.  c /etc/issue
[root@centos8 data]# vim /etc/issue
[root@centos8 data]# rpm -V centos-release 
.......T.  c /etc/issue
```

## 6.yum

### 6.1 yum工作原理

 先在yum服务器上创建 yum repository（仓库），在仓库中事先存储了众多rpm包，以及包的相关的 元数据文件（放置于特定目录repodata下），当yum客户端利用yum/dnf工具进行安装时包时，会自动 下载repodata中的元数据，查询远数据是否存在相关的包及依赖关系，自动从仓库中找到相关包下载并 安装。

yum:Yellowdog Update Modifier

yum服务器的仓库可以多种形式存在：

- file:// 本地路径
- http:// 、https://
- ftp://

### 6.2 yum客户端配置

yum客户端配置文件：

```bash
- /etc/yum.conf         		  #为所有仓库提供公共配置
- /etc/yum.repos.d/*.repo：     #为每个仓库的提供配置文件
- 帮助参考： man 5 yum.conf
```

yum的repo配置文件中可用的变量：

```bash
- $releasever: 当前OS的发行版的主版本号，如：8，7，6
- $arch: CPU架构，如：aarch64, i586, i686，x86_64等
- $basearch：系统基础平台；i386, x86_64
- $contentdir：表示目录，比如：centos-8，centos-7
- $YUM0-$YUM9:自定义变量
```

范例：为centos8配置其他两个常用源（epel和Extras源）

```bash
[root@centos8 yum.repos.d]# vim CentOS-epel.repo
[epel]                                                      
name=Extra Packages for Enterprise Linux 8 - $basearch
baseurl=http://mirrors.cloud.aliyuncs.com/epel/8/Everything/$basearch
failovermethod=priority
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-8

[extras]
name=CentOS-$releasever - Extras
baseurl=http://mirrors.cloud.aliyuncs.com/$contentdir/$releasever/extras/$basearch/os/
gpgcheck=1
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
```

**yum-config-manager命令**

用于管理yum仓库的配置文件

格式：

```bash
[root@centos8 ~]# yum provides yum-config-manager
Last metadata expiration check: 1:31:04 ago on Thu 21 Jan 2021 10:29:03 AM CST.
yum-utils-4.0.17-5.el8.noarch : Yum-utils CLI compatibility layer
Repo        : BaseOS
Matched from:
Filename    : /usr/bin/yum-config-manager

#增加仓库
yum-config-manager --add-repo URL或file
#禁用仓库
yum-config-manager --disable “仓库名"
#启用仓库
yum-config-manager --enable “仓库名” 
```

### 6.3 yum命令

格式：

```bash
yum [options] <command> [<args>...]
```

常见选项：

- -y #自动回答为“yes”
- -q #静默模式
- --nogpgcheck #不进行gpg check
- --enablerepo=repoidglob #临时启用此处指定的repo，支持通配符，如：”*“
- --disablerepo=repoidglob #临时禁用此处指定的repo,和上面语句同时使用，放在后面的生效

##### 6.3.1 显示

格式：

```bash
yum repolist [all|enabled|disabled]						#显示仓库列表,all会列出没启用的仓库
yum list [all | glob_exp1] [glob_exp2] [...]			#显示程序列表,all会列出没安装的包
```

范例：

```bash
[root@centos8 ~]# yum repolist 
repo id                                      repo name
AppStream                                    CentOS-8 - AppStream
BaseOS                                       CentOS-8 - Base
data                                         created by dnf config-manager from file:///data
epel                                         Extra Packages for Enterprise Linux 8 - x86_64
extras                                       CentOS-8 - Extras

[root@centos8 ~]# yum repolist all
repo id             repo name                   		status                                   
AppStream           CentOS-8 - AppStream         		enabled                                   
AppStream-source    CentOS-8 - AppStream Sources    	disabled   

[root@centos8 ~]# yum list all tree
Last metadata expiration check: 2:21:49 ago on Thu 21 Jan 2021 10:29:03 AM CST.
Installed Packages
tree.x86_64            1.7.0-15.el8                                                @anaconda
[root@centos8 ~]# yum list all httpd
Last metadata expiration check: 2:22:00 ago on Thu 21 Jan 2021 10:29:03 AM CST.
Available Packages
httpd.x86_64           2.4.37-30.module_el8.3.0+561+97fdbbcc                       AppStrea
```

##### 6.3.2 安装程序

格式：

```bash
yum install package1 [package2] [...]
yum reinstall package1 [package2] [...] #重新安装
```

范例：

```bash
[root@centos8 data]# yum install [url]
							Install a package directly from a URL
[root@centos8 data]# yum -y install tree
...
Installed:
  tree-1.7.0-15.el8.x86_64                                                       
Complete!
```

##### 6.3.3 卸载程序包

格式：

```bash
yum remove | erase package1 [package2] [...]
```

##### 6.3.4 升级和降级

格式：

```bash
yum update [package1] [package2] [...]
yum downgrade package1 [package2] [...] (降级)
yum check-update(检查可用升级)
```

范例：

```bash
[root@centos7 ~]#yum install samba --disablerepo=updates
[root@centos7 ~]#yum update samba

#更新所有可以更新的软件
[root@centos7 ~]# yum update
```

##### 6.3.5 查询

格式：

```bash
#查看程序包information：
yum info [...]  

#查看指定的特性(可以是某文件)是由哪个程序包所提供：
yum provides | whatprovides feature1 [feature2] [...]

#查看指定包所依赖的capabilities：
yum deplist package1 [package2] [...]
```

范例：

```bash
#注意要写文件全路径才能查询到
[root@centos8 data]# yum provides vsftpd.conf
Last metadata expiration check: 2:51:12 ago on Thu 21 Jan 2021 01:31:05 PM CST.
Error: No Matches found

[root@centos8 data]# yum provides vsftp.conf
Last metadata expiration check: 2:50:03 ago on Thu 21 Jan 2021 01:31:05 PM CST.
Error: No Matches found
[root@centos8 data]# yum provides /etc/vsftpd/vsftpd.conf
Last metadata expiration check: 2:50:58 ago on Thu 21 Jan 2021 01:31:05 PM CST.
vsftpd-3.0.3-32.el8.x86_64 : Very Secure Ftp Daemon
Repo        : AppStream
Matched from:
Filename    : /etc/vsftpd/vsftpd.conf

[root@centos8 data]# yum info tree
Last metadata expiration check: 2:52:35 ago on Thu 21 Jan 2021 01:31:05 PM CST.
Installed Packages
Name         : tree
Version      : 1.7.0
Release      : 15.el8
Architecture : x86_64
Size         : 109 k
Source       : tree-1.7.0-15.el8.src.rpm
Repository   : @System
From repo    : BaseOS
Summary      : File system tree viewer
URL          : http://mama.indstate.edu/users/ice/tree/
License      : GPLv2+
Description  : The tree utility recursively displays the contents of directories in a
             : tree-like format.  Tree is basically a UNIX port of the DOS tree
             : utility.

[root@centos8 data]# yum deplist httpd
Last metadata expiration check: 2:54:15 ago on Thu 21 Jan 2021 01:31:05 PM CST.
package: httpd-2.4.37-30.module_el8.3.0+561+97fdbbcc.x86_64
  dependency: /bin/sh
   provider: bash-4.4.19-12.el8.x86_64
  dependency: /etc/mime.types
   provider: mailcap-2.1.48-3.el8.noarch
  dependency: httpd-filesystem
...
```

##### 6.3.5 仓库缓存

格式：

```bash
#清除目录/var/cache/yum/缓存
yum clean [ packages | metadata | expire-cache | rpmdb | plugins | all ]

#构建缓存
yum makecache
```

范例：

```bash
[root@centos8 data]# du -sh /var/cache/dnf/
41M	/var/cache/dnf/
[root@centos8 data]# yum clean all 
29 files removed
[root@centos8 data]# du -sh /var/cache/dnf/
1.3M	/var/cache/dnf/
[root@centos8 data]# du -sh /var/cache/dnf/
1.3M	/var/cache/dnf/
[root@centos8 data]# yum makecache 
...
Metadata cache created.
[root@centos8 data]# du -sh /var/cache/dnf/
41M	/var/cache/dnf/
```

##### 6.3.6 查看yum历史

yum 执行安装卸载命令会记录到相关日志中

```bash
#CentOS 7以前版本日志
/var/log/yum.log

#CentOS 8 版本日志
/var/log/dnf.rpm.log
/var/log/dnf.log
```

命令

```bash
yum history [info|list|packages-list|packages-info|summary|addoninfo|redo|undo|rollback|new|sync|stats]
```

范例：

```bash
[root@centos8 data]# yum history
ID     | Command line                  			| Date and time    | Action(s)      | Altered
-----------------------------------------------------------------------------------------------
    22 | -y install tree                        | 2021-01-21 16:07 | Install        |    1   
    21 | remove tree                            | 2021-01-21 16:07 | Removed        |    1   
    20 | -y install yum-utils.noarch            | 2021-01-21 12:03 | I, U           |   13  
    
[root@centos8 ~]#dnf history undo 22 -y
Removed:
 dnf-utils-4.0.2.2-3.el8.noarch              
Complete!
[root@centos8 ~]#dnf history redo 22 -y
```

##### 6.3.7 实现私用yum仓库

范例：

```bash
#该例在内网环境采用光盘挂在方式创建yum源
[root@centos7 os]#mount /data/CentOS-7-x86_64-DVD-2009\(1\).iso /home/centos/7/os/
mount: /dev/loop0 写保护，将以只读方式挂载
[root@centos7 os]#ll
总用量 696
-rw-r--r--  3 root root     14 10月 30 05:14 CentOS_BuildTag
drwxr-xr-x  3 root root   2048 10月 27 00:25 EFI
-rw-rw-r-- 21 root root    227 8月  30 2017 EULA
-rw-rw-r-- 21 root root  18009 12月 10 2015 GPL
drwxr-xr-x  3 root root   2048 10月 27 00:26 images
drwxr-xr-x  2 root root   2048 11月  3 00:17 isolinux
drwxr-xr-x  2 root root   2048 10月 27 00:25 LiveOS
drwxr-xr-x  2 root root 673792 11月  4 19:30 Packages
drwxr-xr-x  2 root root   4096 11月  4 19:35 repodata
-rw-rw-r-- 21 root root   1690 12月 10 2015 RPM-GPG-KEY-CentOS-7
-rw-rw-r-- 21 root root   1690 12月 10 2015 RPM-GPG-KEY-CentOS-Testing-7
-r--r--r--  1 root root   2883 11月  4 19:36 TRANS.TBL

#配置仓库服务器
[root@centos7 os]#mv CentOS7-Base-163.repo CentOS7-Base-163.repo.bak
[root@centos7 os]cat Base.repo
[root@centos7 os]#cat /etc/yum.repos.d/Base.repo 
[BaseOS]
name=Centos7 BaseOS
baseurl=file:///home/centos/7/os/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
[root@centos7 os]#yum repolist
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
源标识                  源名称                      状态
BaseOS                 Centos7 BaseOS             4,070
repolist: 4,070

#安装httpd服务
[root@centos7 os]#yum -y install httpd
[root@centos7 os]#service httpd start
Redirecting to /bin/systemctl start  httpd.service
[root@centos7 ~]#cp -a /home/centos/7/os/* /var/www/html/centos/7/

#这里仓库服务端搭建完毕，修改本机仓库baseurl
[root@centos7 ~]#cat /etc/yum.repos.d/Base.repo 
[BaseOS]
name=Centos7 BaseOS
baseurl=http://172.16.60.243/centos/7/
[root@centos7 os]#yum repolist
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
源标识                  源名称                      状态
BaseOS                 Centos7 BaseOS             4,070
repolist: 4,070

#客户端配置yum文件测试
[root@localhost yum.repos.d]# yum -y install vim
...
作为依赖被安装:
  gpm-libs.x86_64 0:1.20.7-6.el7        vim-common.x86_64 2:7.4.629-7.el7        vim-filesystem.x86_64 2:7.4.629-7.el7       
完毕
```

范例：下载extras源，制作私有yum源

```bash
#安装下载工具
[root@centos8 ~]# yum -y install yum-utils

#配置epel源
[root@centos8 ~]# cat /etc/yum.repos.d/CentOS-Base.repo
...
[epelOS]
name=CentOS-$releasever - epel
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=BaseOS&infra=$infra
baseurl=https://mirrors.aliyun.com/epel/8/Everything/x86_64/
gpgcheck=0
enabled=1

#使用工具下载rpm包和metadata
[root@centos8 ~]# yum reposync --repoid=epelOS --download-metadata -p /data

#下载完成将包和metadata拷贝至http中www文件夹下，在按照之前方式配置epel仓库即可
```

## 7.编译安装

### 7.1 c语言源码编译过程

- ./configure

  (1) 通过选项传递参数，指定安装路径、启用特性等；执行时会参考用户的指定以及Makefile.in文 件生成Makefile

  (2) 检查依赖到的外部环境，如依赖的软件包

- make 根据Makefile文件，会检测依赖的环境，进行构建应用程序

- make install 复制文件到相应路径

##### 7.1.1 准备

解决安装环境和依赖

- 开发工具：make, gcc (c/c++编译器GNU C Complier)
- 软件相关依赖包

##### 7.1.2 编译安装

第一步：运行 configure 脚本，生成Makefile 文件

获取其支持使用的选项

- ./configure --help
- --prefix=/PATH：指定默认安装位置,默认为/usr/local/
- --sysconfdir=/PATH：配置文件安装位置

第二步：make

第三步：make install

##### 7.1.3 安装完成配置

1. 二进制程序目录导入至PATH环境变量中

   编辑文件/etc/profile.d/NAME.sh

2. 导入帮助手册

   编辑/etc/man.config|man_db.conf文件,添加一个MANPATH

范例：编译安装 cmatrix

```bash
#准备：系统如果没有相关依赖需先进行安装
#gcc make autoconf ncurses-devel 

#1.下载cmatrix
https://github.com/abishekvashok/cmatrix/releases

#2.解压
[root@centos8 data]# tar -xf cmatrix-v2.0-Butterscotch.tar -C /usr/local/

#3.配置安装路径
[root@centos8 /]# cd /usr/local/src/cmatrix/
[root@centos8 cmatrix]#./configure --prefix=/apps/cmatrix

#4.编译并安装
[root@centos8 cmatrix]#make && make install 

#5.配置环境
[root@centos8 ~]#echo 'PATH=/apps/cmatrix/bin:$PATH' > /etc/profile.d/cmatrix.sh
[root@centos8 ~]#. /etc/profile.d/cmatrix.sh

#或者用软链接实现
[root@centos8 ~]#ln -sv /apps/cmatrix/bin/cmatrix /usr/local/bin/

#6.实现man帮助
[root@centos8 ~]#vim /etc/man_db.conf
MANDATORY_MANPATH                       /apps/cmatrix/share/man
[root@centos8 ~]#man cmatrix
```

范例：编译安装 httpd

```bash
#1.下载安装包解压
[root@localhost src]# tar -xf httpd-2.4.46.tar.bz2

#2.安装依赖
[root@Centos7 httpd-2.4.46]# yum -y install pcre-devel.x86_64
[root@Centos7 src]# tar -xf apr-1.7.0.tar.bz2 -C /usr/local/src/httpd-2.4.46/srclib/
[root@Centos7 src]# tar -xf apr-util-1.6.1.tar.bz2 -C /usr/local/src/httpd-2.4.46/srclib/
[root@Centos7 src]yum -y install expat-devel.x86_64

#3.配置
[root@Centos7 httpd-2.4.46]# ./configure --prefix=/apps/httpd2.4 --with-included-apr

#4.编译安装
[root@Centos7 httpd-2.4.46]#make && make install 

#5.环境配置
[root@Centos7 httpd2.4]#ln -sv /apps/httpd2.4/bin/apachectl /usr/bin/apachectl

#6.关闭防火墙启动服务
[root@Centos7 /]# systemctl stop firewalld.service
[root@Centos7 httpd2.4]#apachectl start
```











