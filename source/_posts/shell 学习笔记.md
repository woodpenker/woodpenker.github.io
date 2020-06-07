---
title: shell学习笔记
tags: [shell,linux]
categories: linux
---
 shell学习笔记
===
  1.  检测变量是否存在：`${varname? ERR:something}`若varname变量未定义则报错后面错误信息"ERR:..."
  
  2.  bash4.0 新特性: `{1..10..2} `从1自增到10每次加2
<!--more-->  
  3.  `d{a,b,c}`代表da,db,dc
  
  4.  `~+`代表PWD值/`~-`代表LODPWD值
  
  5. bash 没有设置-f会支持文件名扩展类似正则:`* ? [ ]`
  
  6. bash 提示符带有颜色: `"\e[0;34m ...  \e[m"`   
  `\e[ 指示开始 0;34m代表颜色代码 \e[m 指示结束`
```
   0;30 黑色 
   0;34蓝色 
   0;32绿色 
   0;36 青色 
   0;31 红色 
   0;35 紫色 
   0;33 褐色
```
   
  7.` tput` 命令设置提示符 `bold`粗体 `rev`反转 `sgr0`关闭所有属性 `setaf `设置前色` setab` 设置背景色 0黑 1红 2绿 3黄 4蓝 5洋红 6青 7白
  
  8. `set -o `开启bash特性 +o关闭
  
  9. linux不允许给目录创建硬链接` ln `不能对目录使用,只能加`-s`创建软连接
  
  10. `/dev/null`是linux的特殊dev文件,是一个黑洞,不想要的东西都可以写进这里.它读不出任何数据.
  
  11. `tar`可以加`--wildcards`与`-xvf`提取制定模式的一组文件或目录如 `--wildcards '*.jpg'`
  
  12. `lsof`或`fuser`可以查看mount的文件系统被哪个进程占用
  
  13. `atq`与`at -l`功能相同
  
  14. `nohup command [arg]  & ` 运行一个对挂起免疫的后台任务
  
  15. bash间接扩展参数:`${!B} `如果B的值是某个变量参数A的名字,那么该式值是参数A的值而不是B的值
  
  16. bash内部变量:
```
      a. $BASH bash全路径; 
      b. $IFS 内部字段分隔符,默认值:"<space><tab><newline>";空格 tab 新行 ; 
      c. $OSTYPE 操作系统类型 ;
      d. $SECONDS 脚本已运行秒数;
      e. $TMOUT 如非0则作为命令read的默认超时秒数; 
      f. $UID 当前用户账号ID
      g. $LINENO shell脚本当前行号
      h. $FUNCNAME 当前执行调用堆栈的所有shell函数名数组,${FUNCNAME[0]}代表当前正在执行shell函数名称,${FUNCNAME[1]}代表调用			${FUNCNAME[0]}的函数的名字
      i. $PS4 bash -x 时显示的默认+号值
```

  17. bash参数 
```
      a. $1 $2 ... 10以上是用{} 即${10} 
      b. $* 代表$1c$2c.... c是IFS的第一个字符,默认空白 常用用法: for arg in $*
      c. "$@" 等价与 "$1" "$2"...每个都是分割的单词 常用用法: for arg in "$@"
      d. $&#35; 参数个数
      e. $! 最后一次执行的后台命令进程号
      f. $0 shell脚本名称
      g. $? 最近一次前台执行命令的退出状态
```

  18. declare 申明变量 -i 代表整型 -r 只读 -p 显示属性和值 -a 申明数组 
  
  19. 数组也可又`$arr=(a1 b2 c3)`定义,引用数组必须加{}, @ *表示数组所有成员,如 `${arr[2]} /${arr[*]} `.不指定索引默认第一个元素 $arr值是a1
  
  20. 数字常量:0x开头表示16进制, 0开头表示8进制 , `BASE&#35;NUMBER `表示BASE进制的数NUMBER 如`12&#35;234` 表示12进制的234
  
  21. let 进行算数运算` let "i=i + 5" `
  
  22. expr进行表达式求值并输出
  
  23. shell返回的状态码0~255间整数,shell每行代码不多于80字符
  
  24. [[]]仅在bash /zsh/ korn shell 中可用,[]在所有符合POSIX标准的shell下可用 
  
  25. test命令:
```
	1）判断表达式
　　if test  (表达式为真)
　　if test !表达式为假
　　test 表达式1 –a 表达式2                  两个表达式都为真
　　test 表达式1 –o 表达式2                 两个表达式有一个为真
　　2）判断字符串
　　test –n 字符串                                   字符串的长度非零
　　test –z 字符串                                    字符串的长度为零
　　test 字符串1＝字符串2                    字符串相等
　　test 字符串1！＝字符串2               字符串不等
　　3）判断整数
　　test 整数1 –eq 整数2                        整数相等
　　test 整数1 –ge 整数2                        整数1大于等于整数2
　　test 整数1 –gt 整数2                         整数1大于整数2
　　test 整数1 –le 整数2                         整数1小于等于整数2
　　test 整数1 –lt 整数2                          整数1小于整数2
　　test 整数1 –ne 整数2                        整数1不等于整数2
　　4）判断文件
　　test  File1 –ef  File2                            两个文件具有同样的设备号和i结点号
　　test  File1 –nt  File2                            文件1比文件2 新
　　test  File1 –ot  File2                            文件1比文件2 旧
　　test –b File                                           文件存在并且是块设备文件
　　test –c File                                           文件存在并且是字符设备文件
　　test –d File                                           文件存在并且是目录
　　test –e File                                           文件存在
　　test –f File                                            文件存在并且是正规文件
　　test –g File                                           文件存在并且是设置了组ID
　　test –G File                                           文件存在并且属于有效组ID
　　test –h File                                           文件存在并且是一个符号链接（同-L）
　　test –k File                                           文件存在并且设置了sticky位
　　test –b File                                           文件存在并且是块设备文件
　　test –L File                                           文件存在并且是一个符号链接（同-h）
　　test –o File                                           文件存在并且属于有效用户ID
　　test –p File                                           文件存在并且是一个命名管道
　　test –r File                                            文件存在并且可读
　　test –s File                                           文件存在并且是一个套接字
　　test –t FD                                             文件描述符是在一个终端打开的
　　test –u File                                           文件存在并且设置了它的set-user-id位
　　test –w File                                          文件存在并且可写
　　test –x File                                           文件存在并且可执行
```
  26. `break [n] `n表示跳出n层嵌套循环.没有n或n小于1,退出码0,否则退出码为n
  
  27. linux下正在运行的进程,会在/proc下存在以进程号命名的子目录,`/proc/[PID]/fd `的每一个条目对应一个该进程打开的文件,用文件描述符命名,并软连接到实际文件
  
  28. 函数  

	1. 创建函数一定要在调用函数之前
	2. function_name() 或者 function name        //function是关键字 name 后面可以省略()
	{
	commands...
	[return int;] 返回值0~255 没有则默认返回最后一条指令运行返回值
	}
	3. 函数自己使用$1 $2 ... 作为传递的参数 $* $@ 代表所有的传递的参数 $&#35;传递给函数参数个数
	4. shell默认情况下所有变量都是全局变量,函数内可使用local定义内部变量,local只能在函数体内使用
	5. 在脚本中加载函数文件中的函数 使用 点好. 或者source如: . /path/function.sh 或source /path/function.sh
	6. 调用函数直接输入名称,不加() 
	7. &可以将函数放在后台运行

  29. bash正则
	```
	*： 代表 0 个或者多个任意字符
	?： 代表一定有一个的任意字符
	[]： 代表一定有一个在括号内的字符（非任意字符）。例如[abcd]代表一定有一个字符，可能是 abcd 这四个选项的任意一个。
	[-]：若邮件韩在括号内时，代表在编码顺序内的所有自负。例如：[0-9]代表 0 到 9 之间的所有数字，因为数字的语系编码是连续的。
	[^]： 若括号内的第一个字符为指数字符(^)，那表示反向选择，例如：[^abc]代表一定有一个字符，只要是非 abc 的其他字符就可以。
	^：匹配行首位置
	$：匹配行尾位置
	.：匹配任意祖父
	*：对*之前的匹配整体或字符匹配任意次（包括 0 次）
	\?：对\?之前的匹配整体或字符匹配 0 次或 1 次
	\{n\}: 对 \ { 之前的匹配整体或字符匹配 n 次
	\{m,\}: 对 \ { 之前的匹配整体或字符匹配至少 m 次
	\{m,n}: 对 \ { 之前的匹配整体或字符匹配 m 到 n 次
	\<\>匹配单词边界,如:\<the\>只匹配 the 不匹配then
	[abcdef]: 对单字符而言匹配[]中的字符
	[a-z]： 对单字符而言，匹配任意一个小写字母
	[^a-z]：不匹配括号中的内容
	+ :重复『一个或一个以上』的前一个 RE 字符,与*类似 但不包括0个字符的情况
	?：『零个或一个』的前一个 RE 字符
	|：用或( or )的方式找出数个字串
	()：找出『群组』字串与| 一起使用
	()+：多个重复群组的判别
	```

	POSIX字符类:
	```
	[:alnum:]	字母数字字符
	[:alpha:]	字母字符
	[:cntrl:]	控制字符
	[:digit:]	数字字符
	[:graph:]	非空白字符(非空格、控制字符等)
	[:lower:]	小写字母
	[:print:]	与[:graph:]相似，但是包含空格字符
	[:punct:]	标点字符
	[:space:]	所有的空白字符(换行符、空格、制表符)
	[:upper:]	大写字母
	[:xdigit:]	允许十六进制的数字(0-9a-fA-F)
	```
 
  30. 关闭脚本case语句对参数大小写敏感:在case前加入语句shopt -s nocasematch 重新打开用 shopt -u nocasematch
  
  31. shift [n] 命令向左移动参数变量,n代表每次移动位数, shift后一般加对$?的值做判断语句来检查shift是否被执行
  
  32. getopts (bash内置)的
```
      a. 使用形式是：getopts option_string variable.getopts一共有两个参数，第一个是-a这样的选项，第二个参数是 hello这样的参数。
      b. 选项之间可以通过冒号:进行分隔，也可以直接相连接，：表示选项后面必须带有参数，如果没有可以不加实际值进行传递.
      c. 当optstring以”:”开头时，getopts会区分invalid option错误和miss option argument错误。
      d. 如果optstring不以”:“开头，invalid option错误和miss option argument错误都会使varname被设成?，$OPTARG是出问题的option。 
```
  33. getopt(命令行命令) 命令的选项说明：
```
-a 使getopt长参数支持"-"符号打头，必须与-l同时使用
-l 后面接getopt支持长参数列表.选项用逗号分割,":"表示选项需要一个参数,"::"表示选项有个可选参数
-n program如果getopt处理参数返回错误，会指出是谁处理的这个错误，这个在调用多个脚本时，很有用
-o 后面接短参数列表，这种用法与getopts类似.每个字符代表一个选项,":"表示选项需要一个参数,"::"表示选项有个可选参数
-u 不给参数列表加引号，默认是加引号的（不使用-u选项），例如在加不引号的时候 --longopt "select * from db1.table1" $2只会取到select ，而不是完整的SQL语句。
```
使用eval 的目的是为了防止参数中有shell命令，被错误的扩展。
样例:
```
              ARGV=($(getopt -o 短选项1[:]短选项2[:]...[:]短选项n -l 长选项1,长选项2,...,长选项n -- "$@"))
              eval set -- "$ARGV"
              while true
               do
                  case "$1" in
                   -短选项1|--长选项1)
                       process
                      shift
                      ;;
                  -短选项2|--长选项2)
                       &#35; 获取选项
                       opt = $2
                       process
                      shift 2
                      ;;
                   ......
                  -短选项3|--长选项3)
                       process
                       ;;
                   --)
               break
              ;;
               esac
               }
```
  34. read 读入用户输入 -p 在输入前显示信息 -t 输入超时设置 -s隐藏用户输入 
 
  35. 重定向:
      a. cat将输出接管道|进行输入数据的方法会不好,最好让你的程序从标准输入读取数据用<,这样用户就可以使用重定向来获取数据.
      b. 从文本输入用<<EOF格式,结束标识是EOF,必须写在行首.
      c. <<<用于输入重定向的普通字符串,后面接的就是要输入的字符
      d. 0 输入 1 输出 2错误 
      e. 同时重定向 "&>" ">&" "2>&1" 追加用 >> 
      f. 在shell中最多可以有9个打开的文件描述符0-8.exec 用于操作文件描述符,如exec 2>err.log.符号<&可以复制一个输入文件描述符，符号>&可以复制一个输出描述符。exec n<&- 关闭文件描述符.exec [n] <>file 打开读写文件
      
  36. 管道|与重定向的区别是,管道是将第一个命令的输出作为第二个的输入,而重定向是将命令与文件链接.
  
  37. 常用的管道过滤命令: awk,cut, grep, tar , head 取文件开头,paste 合并文件行, sed ,sort 行排序, split 文件分割, strings 打印文件中可打印的字符串, tac 与cat相反 倒序显示文件,tail 显示文件结尾 , tee 从标准输入读取内容并写入标准文件或输出, tr 转换或删除字符, uniq 报告或忽略重复的行, wc 统计行数字数等,column进行输出的表格格式化.
  
  38. posix信号:	
   在linux下的:       
```
   1) SIGHUP	 2) SIGINT	 3) SIGQUIT	 4) SIGILL	 5) SIGTRAP
 6) SIGABRT	 7) SIGBUS	 8) SIGFPE	 9) SIGKILL	10) SIGUSR1
11) SIGSEGV	12) SIGUSR2	13) SIGPIPE	14) SIGALRM	15) SIGTERM
16) SIGSTKFLT	17) SIGCHLD	18) SIGCONT	19) SIGSTOP	20) SIGTSTP
21) SIGTTIN	22) SIGTTOU	23) SIGURG	24) SIGXCPU	25) SIGXFSZ
26) SIGVTALRM	27) SIGPROF	28) SIGWINCH	29) SIGIO	30) SIGPWR
31) SIGSYS	34) SIGRTMIN	35) SIGRTMIN+1	36) SIGRTMIN+2	37) SIGRTMIN+3
38) SIGRTMIN+4	39) SIGRTMIN+5	40) SIGRTMIN+6	41) SIGRTMIN+7	42) SIGRTMIN+8
43) SIGRTMIN+9	44) SIGRTMIN+10	45) SIGRTMIN+11	46) SIGRTMIN+12	47) SIGRTMIN+13
48) SIGRTMIN+14	49) SIGRTMIN+15	50) SIGRTMAX-14	51) SIGRTMAX-13	52) SIGRTMAX-12
53) SIGRTMAX-11	54) SIGRTMAX-10	55) SIGRTMAX-9	56) SIGRTMAX-8	57) SIGRTMAX-7
58) SIGRTMAX-6	59) SIGRTMAX-5	60) SIGRTMAX-4	61) SIGRTMAX-3	62) SIGRTMAX-2
63) SIGRTMAX-1	64) SIGRTMAX	
信号	     	取值		默认动作	含义（发出信号的原因）
SIGHUP		1	Term	终端的挂断或进程死亡
SIGINT		2	Term	来自键盘的中断信号
SIGQUIT		3	Core	来自键盘的离开信号
SIGILL		4	Core	非法指令
SIGABRT	6	Core	来自abort的异常信号
SIGFPE		8	Core	浮点例外
SIGKILL		9	Term	杀死
SIGSEGV	11	Core	段非法错误(内存引用无效)
SIGPIPE		13	Term	管道损坏：向一个没有读进程的管道写数据
SIGALRM	14	Term	来自alarm的计时器到时信号
SIGTERM	15	Term	终止
SIGUSR1	30,10,16	Term	用户自定义信号1
SIGUSR2	31,12,17	Term	用户自定义信号2
SIGCHLD	20,17,18	Ign	子进程停止或终止
SIGCONT	19,18,25	Cont	如果停止，继续执行
SIGSTOP	17,19,23	Stop	非来自终端的停止信号
SIGTSTP		18,20,24	Stop	来自终端的停止信号
SIGTTIN		21,21,26	Stop	后台进程读终端
SIGTTOU	22,22,27	Stop	后台进程写终端
　			　
SIGBUS		10,7,10	Core	总线错误（内存访问错误）
SIGPOLL		Term	Pollable	事件发生(Sys V)，与SIGIO同义
SIGPROF	27,27,29	Term	统计分布图用计时器到时
SIGSYS		12,-,12	Core	非法系统调用(SVr4)
SIGTRAP	5		Core	跟踪/断点自陷
SIGURG		16,23,21	Ign	socket紧急信号(4.2BSD)
SIGVTALRM	26,26,28	Term	虚拟计时器到时(4.2BSD)
SIGXCPU	24,24,30	Core	超过CPU时限(4.2BSD)
SIGXFSZ		25,25,31	Core	超过文件长度限制(4.2BSD)
　			　
SIGIOT		6		Core	IOT自陷，与SIGABRT同义
SIGEMT		7,-,7		Term
SIGSTKFLT	-,16,-	Term	协处理器堆栈错误(不使用)
SIGIO		23,29,22	Term	描述符上可以进行I/O操作
SIGCLD		-,-,18	Ign	与SIGCHLD同义
SIGPWR		29,30,19	Term	电力故障(System V)
SIGINFO		29,-,-			与SIGPWR同义
SIGLOST	-,-,-		Term	文件锁丢失
SIGWINCH	28,28,20	Ign	窗口大小改变(4.3BSD, Sun)
SIGUNUSED	-,31,-	Term	未使用信号(will be SIGSYS)
```
一些信号的取值是硬件结构相关的（一般alpha和sparc架构用第一个值，i386、ppc和sh架构用中间值，mips架构用第三个值， - 表示相应架构的取值未知）。
SIGKILL和SIGSTOP信号不能被挂钩、阻塞或忽略。

  39. disown -h %1 会阻止shell 向后台作业1发送SIGHUP信号,这时退出shell作业依旧会执行,如果用shopt打开了shell内部的huponexit,会在退出shell时默认向所有作业发送SIGHUP
  
  40. 进程的状态:`D 不可中断休眠 R运行 S 休眠 T 停止 Z 僵死`
  
  41. bash下:
  `Ctrl + C` 中断信号,发送SIGINT. 
  `Ctrl+Y` 延时挂起,使运行的进程在尝试从终端读取数据时停止,控制权返回shell.
  `Ctrl+Z` 挂起信号,发送SIGTSTP,返回shell
  
  42. `(command1;command2;...)`内嵌在圆括号内部的命令列表,作为一个shell进程的子shell进程运行,其变量对外不可见,运行环境变量可设置不同.
  
  43. 捕获:
      a. `trap command signal [signal ...] `捕获特定的信号进行处理 如` trap "kill -9 $self " SIGHUP` (在收到sinhup信号后立即杀死自己)
      b. trap语句中若""内为空则不做响应,可用于屏蔽Ctrl+C 等
      c. `trap - signal...` 用于重置默认模式,移除信号的捕获.
      
      
sed手册
---

[参考站点](https://www.gnu.org/software/sed/manual/sed.html)
```
[root@www ~]&#35; sed [-nefr] [动作]
选项与参数：
-n ：使用安静(silent)模式。在一般 sed 的用法中，所有来自 STDIN 的数据一般都会被列出到终端上。但如果加上 -n 参数后，则只有经过sed 特殊处理的那一行(或者动作)才会被列出来。
-e ：直接在命令列模式上进行 sed 的动作编辑；
-f ：直接将 sed 的动作写在一个文件内， -f filename 则可以运行 filename 内的 sed 动作；
-r ：sed 的动作支持的是延伸型正规表示法的语法。(默认是基础正规表示法语法)
-i ：直接修改读取的文件内容，而不是输出到终端。
动作说明： [n1[,n2]]function
n1, n2 ：不见得会存在，一般代表『选择进行动作的行数』，举例来说，如果我的动作是需要在 10 到 20 行之间进行的，则『 10,20[动作行为] 』
function：
a ：新增， a 的后面可以接字串，而这些字串会在新的一行出现(目前的下一行)～
c ：取代， c 的后面可以接字串，这些字串可以取代 n1,n2 之间的行！
d ：删除，因为是删除啊，所以 d 后面通常不接任何咚咚；
i ：插入， i 的后面可以接字串，而这些字串会在新的一行出现(目前的上一行)；
p ：列印，亦即将某个选择的数据印出。通常 p 会与参数 sed -n 一起运行～
s ：取代，可以直接进行取代的工作哩！通常这个 s 的动作可以搭配正规表示法！例如 1,20s/old/new/g 就是啦！
```

awk 
---

[参考站点](https://www.gnu.org/software/gawk/manual/gawk.html)
`awk '{pattern + action}' {filenames}`
尽管操作可能会很复杂，但语法总是这样，其中` pattern `表示 AWK 在数据中查找的内容，而 `action `是在找到匹配内容时所执行的一系列命令。花括号`（{}）`不需要在程序中始终出现，但它们用于根据特定的模式对一系列指令进行分组。 pattern就是要表示的正则表达式，用斜杠括起来。
awk语言的最基本功能是在文件或者字符串中基于指定规则浏览和抽取信息，awk抽取信息后，才能进行其他文本操作。完整的awk脚本通常用来格式化文本文件中的信息。
通常，awk是以文件的一行为处理单位的。awk每接收文件的一行，然后执行相应的命令，来处理文本。
模式可以是以下任意一个：` /正则表达式/`：使用通配符的扩展集。 关系表达式：使用运算符进行操作，可以是字符串或数字的比较测试。 模式匹配表达式：用运算符~（匹配）和~!（不匹配）。 BEGIN语句块、pattern语句块、END语句块.


`set -- `
---

  表示:"If  no  arguments follow this option, then the positional parameters are unset.  Otherwise, the positional parameters are set  to the args, even if some of them begin with a -."

shell 调用其他脚本
---

```
 使用./1.sh 或者 sh 1.sh 执行后使用exit退出并返回 ,
 . 1.sh 或 export 1.sh 执行后用return返回.
 
shell脚本中的函数类似使用./1.sh方式执行的,有输入变量和返回但返回用return

所有shell return只能返回数字,如果需要返回字符串,在func中使用echo.

用"."或者export执行的相当于本脚本执行,变量通用.无须返回.
```

echo和printf命令
---

`echo -e "1\t2\t3" `
显示结果为`123`
echo 会自动在输出结尾加上一个\n回车,但是printf不会。
echo的-e命令表示需要转义，把\t转换成tab空格，-n命令会忽略输出结尾的回车。

***echo打印彩色输出：*** `echo -e "\e[1;31m this is red \e[0m"`
其中\e[1;31m表示颜色开始，\e[0m表示重置即结束。31代表红色，修改数字对应不同颜色：黑色30，红色31，绿色32，黄色33，蓝色34，洋红35，青色36，白色37。


print命令和c语言的类似，
`printf "%-5s %-6.2f" hello 43.2222`表示输出`hello 43.22`，
-表示左对齐，否组默认右对齐，数字是格式化位数。

`var=value`和`var = value` 不同，前者是赋值，后者是等于比较操作,注意中间的空格区分。

***获取字符串长度***
采用`${&#35;var}` var表示所要获取的字符串
例如：
```bash
var=1234567890
echo ${&#35;var}

```
输出：`>10`

