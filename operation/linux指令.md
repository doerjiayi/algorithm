
##head和tail   cat filename | tail -n +3000 | head -n 1000
情景linux–一张图搞懂head -n和tail -n
https://blog.csdn.net/signjing/article/details/69357769

linux取出某几行 
一、显示3000~3999行
cat filename | tail -n +3000 | head -n 1000
二、显示1000行到3000行
cat filename| head -n 3000 | tail -n +1000
注意两种方法的1653顺序
分解：
tail -n 1000：显示最后1000行
tail -n +1000：从1000行开始显示，显示1000行以后的
head -n 1000：显示前面1000行
三、用sed命令
sed -n '5,10p' filename 这样就可以只查看文件的第5行到第10行。 

##tee ls | tee out.txt
https://blog.csdn.net/weixin_34362790/article/details/86443986

tee命令用于将数据重定向到文件，另一方面还可以提供一份重定向数据的副本作为后续命令的stdin。简单的说就是把数据重定向到给定文件和屏幕上。

存在缓存机制，每1024个字节将输出一次。若从管道接收输入数据，应该是缓冲区满，才将数据转存到指定的文件中。若文件内容不到1024个字节，则接收完从标准输入设备读入的数据后，将刷新一次缓冲区，并转存数据到指定文件。

语法
tee(选项)(参数)
选项
-a：向文件中重定向时使用追加模式；
-i：忽略中断（interrupt）信号。
参数
文件：指定输出重定向的文件。
在终端打印stdout同时重定向到文件中：
ls | tee out.txt
1.sh
1.txt
2.txt
eee.tst
EEE.tst
one
out.txt
string2
www.pdf
WWW.pdf
WWW.pef
[root@localhost text]# ls | tee out.txt | cat -n
     1  1.sh
     2  1.txt
     3  2.txt
     4  eee.tst
     5  EEE.tst
     6  one
     7  out.txt
     8  string2
     9  www.pdf
    10  WWW.pdf
    11  WWW.pef



##shell之三大文本处理工具grep、sed及awk 
https://www.cnblogs.com/-zyj/p/5763303.html

grep：文本过滤器，如果仅仅是过滤文本，可使用grep，其效率要比其他的高很多；
sed：Stream EDitor，流编辑器，默认只处理模式空间，不处理原数据，如果你处理的数据是针对行进行处理的，可以使用sed；
awk：报告生成器，格式化以后显示。如果对处理的数据需要生成报告之类的信息，或者你处理的数据是按列进行处理的，最好使用awk。
 
###grep grep '[a-z]\{5}\' aa
grep(关键字: 截取) 文本搜集工具, 结合正则表达式非常强大
主要参数 []
-c : 只输出匹配的行
-I : 不区分大小写
-h : 查询多文件时不显示文件名
-l : 查询多文件时, 只输出包含匹配字符的文件名
-n : 显示匹配的行号及行
-v : 显示不包含匹配文本的所有行(我经常用除去grep本身)
基本工作方式: grep 要匹配的内容 文件名, 例如:
       grep 'test' d* 显示所有以d开头的文件中包含test的行
       grep 'test' aa bb cc 显示在 aa bb cc 文件中包含test的行
       grep '[a-z]\{5}\' aa 显示所有包含字符串至少有5个连续小写字母的串
上文已经做出说明
        http://www.cnblogs.com/-zyj/p/5760484.html

###sed
sed(关键字: 编辑) 以行为单位的文本编辑工具 sed可以直接修改档案, 不过一般不推荐这么做, 可以分析 standard input
基本工作方式: sed [-nef] '[动作]' [输入文本]
          a\ ： 在当前行后添加一行或多行。多行时除最后一行外，每行末尾需用“\”续行
　　    c\ ：用此符号后的新文本替换当前行中的文本。多行时除最后一行外，每⾏末尾需用”\"续行
　  　  i\ ：在当前行之前插入文本。多行时除最后一行外，每行末尾需用”\"续行删除行
　　    h ： 把模式空间里的内容复制到暂存缓冲区
　　    H ： 把模式空间里的内容追加到暂存缓冲区
　　   g ： 把暂存缓冲区里的内容复制到模式空间，覆盖原有的内容
　　   G： 把暂存缓冲区的内容追加到模式空间⾥，追加在原有内容的后面
　   　l ： 列出非打印字符
　　   p ： 打印行
　　   q ： 结束或退出sed
　　   r ： 从文件中读取输入行
　   　! ： 对所选行以外的所有行应用命令
　  　 s ： 用一个字符串替换另一个
　 　  g ： 在行内进行全局替换
　     w ： 将所选的行写入文件
　　   x ： 交换暂存缓冲区与模式空间的内容
　 　  y ： 将字符替换为另一字符（不能对正则表达式使用y命令）
选项
　　-e ： 进行多项编辑，即对输入行应用多条sed命令时使用
　　-n ： 取消默认的输出
　　-f ：指定sed脚本的文件名

###awk
 
sed以行为单位处理文件,awk比sed强的地方在于不仅能以行为单位还能以列为单位处理文件。 awk缺省的行分隔符是换行,缺省的列分隔符是连续的空格和Tab,
但是行分隔符和列分隔符都可以自定义,比如/etc/passwd文件的每一行有干个字段,字段之间以:分隔,就可以重新定义awk的列分隔符为:并以列为单位处理这个文件。
awk实际上是一门很复杂的脚本语言,还有像C语言一样的分支和循环结构,但是基本语法和sed类似,awk命令行的基本形式为:
　　awk option 'script' file1 file2 ... 
　　awk option -f scriptfile file1 file2 ...
　　和sed一样,awk处理的文件既可以由标准输入重定向得到,也可以当命令行参数传入,编辑命令可以直接当命令行参数传入,也可以用-f参数指定一个脚本文件,
编辑命令的格式为:
　　/pattern/{actions}
　　和sed类似,pattern是正则表达式,actions是一系列操作。 awk程序一行一行读出待处理文件,如果某一行与pattern匹配,或者满足condition条件,
则执行相应的actions,如果一条awk命令只有actions部分,则actions作用于待处理文件的每一行。

注：
$0:表示当前行
$1:表示当前行的第一列
$2:表示当前行的第二列

##find find . -type f -name '*.log*' | xargs grep --color -n 'Identify'

find grep 联合使用 过滤所有子目录、文件
find . -type f -name '*.log*' | xargs grep --color -n 'GetConnectIdentify'


##Linux wc命令详解  cat /etc/passwd | wc -l
wc常见命令参数 
wc  -l : 统计行
wc  -c: 统计字节数
wc  -m:统计字符数，不能与-c同时使用
wc  -w:统计字数
wc  -L:打印最长长度
注意： wc 可以直接后面跟文件使用，但是会显示文件 ls -l|wc -l 统计行的时候包含了当前目录，所以会多一个
常用的命令展示

常见的统计行数的命令 
wc -l h.txt
cat -n h.txt | tail -n 
sed '=' h.txt  | awk '{print $1}'
awk '{print NR $0}' h.txt  | tail -1
 
grep -n "." h.txt
查看/etc/passwd有多少用户 

 统计行数
[root@localhost omc]# cat /etc/passwd | wc -l

 统计最长
[root@localhost omc]# cat /etc/passwd | wc -L

##lsof
https://www.cnblogs.com/sparkbj/p/7161669.html

简介
lsof(list open files)是一个列出当前系统打开文件的工具。在linux环境下，任何事物都以文件的形式存在，通过文件不仅仅可以访问常规数据，还可以访问网络连接和硬件。所以如传输控制协议 (TCP) 和用户数据报协议 (UDP) 套接字等，系统在后台都为该应用程序分配了一个文件描述符，无论这个文件的本质如何，该文件描述符为应用程序与基础操作系统之间的交互提供了通用接口。因为应用程序打开文件的描述符列表提供了大量关于这个应用程序本身的信息，因此通过lsof工具能够查看这个列表对系统监测以及排错将是很有帮助的。
 
输出信息含义
在终端下输入lsof即可显示系统打开的文件，因为 lsof 需要访问核心内存和各种文件，所以必须以 root 用户的身份运行它才能够充分地发挥其功能。
直接输入lsof部分输出为:

COMMAND     PID        USER   FD      TYPE             DEVICE SIZE/OFF       NODE NAME init          1        root  cwd       DIR                8,1     4096          2 / init          1        root  rtd       DIR                8,1     4096          2 / init          1        root  txt       REG                8,1   150584     654127 /sbin/init udevd       415        root    0u      CHR                1,3      0t0       6254 /dev/null udevd       415        root    1u      CHR                1,3      0t0       6254 /dev/null udevd       415        root    2u      CHR                1,3      0t0       6254 /dev/null udevd       690        root  mem       REG                8,1    51736     302589 /lib/x86_64-linux-gnu/libnss_files-2.13.so syslogd    1246      syslog    2w      REG                8,1    10187     245418 /var/log/auth.log syslogd    1246      syslog    3w      REG                8,1    10118     245342 /var/log/syslog dd         1271        root    0r      REG                0,3        0 4026532038 /proc/kmsg dd         1271        root    1w     FIFO               0,15      0t0        409 /run/klogd/kmsg dd         1271        root    2u      CHR                1,3      0t0       6254 /dev/null

每行显示一个打开的文件，若不指定条件默认将显示所有进程打开的所有文件。
lsof输出各列信息的意义如下：
COMMAND：进程的名称 PID：进程标识符
USER：进程所有者
FD：文件描述符，应用程序通过文件描述符识别该文件。如cwd、txt等 TYPE：文件类型，如DIR、REG等
DEVICE：指定磁盘的名称
SIZE：文件的大小
NODE：索引节点（文件在磁盘上的标识）
NAME：打开文件的确切名称
FD 列中的文件描述符cwd 值表示应用程序的当前工作目录，这是该应用程序启动的目录，除非它本身对这个目录进行更改,txt 类型的文件是程序代码，如应用程序二进制文件本身或共享库，如上列表中显示的 /sbin/init 程序。
其次数值表示应用程序的文件描述符，这是打开该文件时返回的一个整数。如上的最后一行文件/dev/initctl，其文件描述符为 10。u 表示该文件被打开并处于读取/写入模式，而不是只读 ® 或只写 (w) 模式。同时还有大写 的W 表示该应用程序具有对整个文件的写锁。该文件描述符用于确保每次只能打开一个应用程序实例。初始打开每个应用程序时，都具有三个文件描述符，从 0 到 2，分别表示标准输入、输出和错误流。所以大多数应用程序所打开的文件的 FD 都是从 3 开始。
与 FD 列相比，Type 列则比较直观。文件和目录分别称为 REG 和 DIR。而CHR 和 BLK，分别表示字符和块设备；或者 UNIX、FIFO 和 IPv4，分别表示 UNIX 域套接字、先进先出 (FIFO) 队列和网际协议 (IP) 套接字。
 
常用参数
lsof语法格式是： lsof ［options］ filename

lsof abc.txt 显示开启文件abc.txt的进程 lsof -c abc 显示abc进程现在打开的文件 lsof -c -p 1234 列出进程号为1234的进程所打开的文件 lsof -g gid 显示归属gid的进程情况 lsof +d /usr/local/ 显示目录下被进程开启的文件 lsof +D /usr/local/ 同上，但是会搜索目录下的目录，时间较长 lsof -d 4 显示使用fd为4的进程 lsof -i 用以显示符合条件的进程情况 lsof -i[46] [protocol][@hostname|hostaddr][:service|port]   46 --> IPv4 or IPv6   protocol --> TCP or UDP   hostname --> Internet host name   hostaddr --> IPv4地址   service --> /etc/service中的 service name (可以不止一个)   port --> 端口号 (可以不止一个)

 
lsof使用实例
查找谁在使用文件系统
在卸载文件系统时，如果该文件系统中有任何打开的文件，操作通常将会失败。那么通过lsof可以找出那些进程在使用当前要卸载的文件系统，如下： # lsof /GTES11/ COMMAND PID USER FD TYPE DEVICE SIZE NODE NAME bash 4208 root cwd DIR 3,1 4096 2 /GTES11/ vim 4230 root cwd DIR 3,1 4096 2 /GTES11/ 在这个示例中，用户root正在其/GTES11目录中进行一些操作。一个 bash是实例正在运行，并且它当前的目录为/GTES11，另一个则显示的是vim正在编辑/GTES11下的文件。要成功地卸载/GTES11，应该在通知用户以确保情况正常之后，中止这些进程。 这个示例说明了应用程序的当前工作目录非常重要，因为它仍保持着文件资源，并且可以防止文件系统被卸载。这就是为什么大部分守护进程（后台进程）将它们的目录更改为根目录、或服务特定的目录（如 sendmail 示例中的 /var/spool/mqueue）的原因，以避免该守护进程阻止卸载不相关的文件系统。
 
恢复删除的文件
当Linux计算机受到入侵时，常见的情况是日志文件被删除，以掩盖攻击者的踪迹。管理错误也可能导致意外删除重要的文件，比如在清理旧日志时，意外地删除了数据库的活动事务日志。有时可以通过lsof来恢复这些文件。 当进程打开了某个文件时，只要该进程保持打开该文件，即使将其删除，它依然存在于磁盘中。这意味着，进程并不知道文件已经被删除，它仍然可以向打开该文件时提供给它的文件描述符进行读取和写入。除了该进程之外，这个文件是不可见的，因为已经删除了其相应的目录索引节点。 在/proc 目录下，其中包含了反映内核和进程树的各种文件。/proc目录挂载的是在内存中所映射的一块区域，所以这些文件和目录并不存在于磁盘中，因此当我们对这些文件进行读取和写入时，实际上是在从内存中获取相关信息。大多数与 lsof 相关的信息都存储于以进程的 PID 命名的目录中，即 /proc/1234 中包含的是 PID 为 1234 的进程的信息。每个进程目录中存在着各种文件，它们可以使得应用程序简单地了解进程的内存空间、文件描述符列表、指向磁盘上的文件的符号链接和其他系统信息。lsof 程序使用该信息和其他关于内核内部状态的信息来产生其输出。所以lsof 可以显示进程的文件描述符和相关的文件名等信息。也就是我们通过访问进程的文件描述符可以找到该文件的相关信息。 当系统中的某个文件被意外地删除了，只要这个时候系统中还有进程正在访问该文件，那么我们就可以通过lsof从/proc目录下恢复该文件的内容。 假如由于误操作将/var/log/messages文件删除掉了，那么这时要将/var/log/messages文件恢复的方法如下： 首先使用lsof来查看当前是否有进程打开/var/logmessages文件，如下： # lsof |grep /var/log/messages syslogd 1283 root 2w REG 3,3 5381017 1773647 /var/log/messages (deleted) 从上面的信息可以看到 PID 1283（syslogd）打开文件的文件描述符为 2。同时还可以看到/var/log/messages已经标记被删除了。因此我们可以在 /proc/1283/fd/2 （fd下的每个以数字命名的文件表示进程对应的文件描述符）中查看相应的信息，如下： # head -n 10 /proc/1283/fd/2 Aug 4 13:50:15 holmes86 syslogd 1.4.1: restart. Aug 4 13:50:15 holmes86 kernel: klogd 1.4.1, log source = /proc/kmsg started. Aug 4 13:50:15 holmes86 kernel: Linux version 2.6.22.1-8 (root@everestbuilder.linux-ren.org) (gcc version 4.2.0) #1 SMP Wed Jul 18 11:18:32 EDT 2007 Aug 4 13:50:15 holmes86 kernel: BIOS-provided physical RAM map: Aug 4 13:50:15 holmes86 kernel: BIOS-e820: 0000000000000000 - 000000000009f000 (usable) Aug 4 13:50:15 holmes86 kernel: BIOS-e820: 000000000009f000 - 00000000000a0000 (reserved) Aug 4 13:50:15 holmes86 kernel: BIOS-e820: 0000000000100000 - 000000001f7d3800 (usable) Aug 4 13:50:15 holmes86 kernel: BIOS-e820: 000000001f7d3800 - 0000000020000000 (reserved) Aug 4 13:50:15 holmes86 kernel: BIOS-e820: 00000000e0000000 - 00000000f0007000 (reserved) Aug 4 13:50:15 holmes86 kernel: BIOS-e820: 00000000f0008000 - 00000000f000c000 (reserved) 从上面的信息可以看出，查看 /proc/8663/fd/15 就可以得到所要恢复的数据。如果可以通过文件描述符查看相应的数据，那么就可以使用 I/O 重定向将其复制到文件中，如: cat /proc/1283/fd/2 > /var/log/messages 对于许多应用程序，尤其是日志文件和数据库，这种恢复删除文件的方法非常有用。
 
可以列出被进程所打开的文件的信息。被打开的文件可以是
1.普通的文件，2.目录  3.网络文件系统的文件，4.字符设备文件  5.(函数)共享库  6.管道，命名管道 7.符号链接
8.底层的socket字流，网络socket，unix域名socket
9.在linux里面，大部分的东西都是被当做文件的…..还有其他很多
怎样使用lsof
这里主要用案例的形式来介绍lsof 命令的使用
1.列出所有打开的文件:
lsof
备注: 如果不加任何参数，就会打开所有被打开的文件，建议加上一下参数来具体定位
2. 查看谁正在使用某个文件
lsof   /filepath/file
3.递归查看某个目录的文件信息
lsof +D /filepath/filepath2/
备注: 使用了+D，对应目录下的所有子目录和文件都会被列出
4. 比使用+D选项，遍历查看某个目录的所有文件信息 的方法
lsof | grep ‘/filepath/filepath2/’
5. 列出某个用户打开的文件信息
lsof  -u username
备注: -u 选项，u其实是user的缩写
6. 列出某个程序所打开的文件信息
lsof -c mysql
备注: -c 选项将会列出所有以mysql开头的程序的文件，其实你也可以写成lsof | grep mysql,但是第一种方法明显比第二种方法要少打几个字符了
7. 列出多个程序多打开的文件信息
lsof -c mysql -c apache
8. 列出某个用户以及某个程序所打开的文件信息
lsof -u test -c mysql
9. 列出除了某个用户外的被打开的文件信息
lsof   -u ^root
备注：^这个符号在用户名之前，将会把是root用户打开的进程不让显示
10. 通过某个进程号显示该进行打开的文件
lsof -p 1
11. 列出多个进程号对应的文件信息
lsof -p 123,456,789
12. 列出除了某个进程号，其他进程号所打开的文件信息
lsof -p ^1
13 . 列出所有的网络连接
lsof -i
14. 列出所有tcp 网络连接信息
lsof  -i tcp
15. 列出所有udp网络连接信息
lsof  -i udp
16. 列出谁在使用某个端口
lsof -i :3306
17. 列出谁在使用某个特定的udp端口
lsof -i udp:55
特定的tcp端口
lsof -i tcp:80
18. 列出某个用户的所有活跃的网络端口
lsof  -a -u test -i
19. 列出所有网络文件系统
lsof -N
20.域名socket文件
lsof -u
21.某个用户组所打开的文件信息
lsof -g 5555
22. 根据文件描述列出对应的文件信息
lsof -d description(like 2)
23. 根据文件描述范围列出文件信息
lsof -d 2-3
 
实用命令


lsof `which httpd` //那个进程在使用apache的可执行文件 lsof /etc/passwd //那个进程在占用/etc/passwd lsof /dev/hda6 //那个进程在占用hda6 lsof /dev/cdrom //那个进程在占用光驱 lsof -c sendmail //查看sendmail进程的文件使用情况 lsof -c courier -u ^zahn //显示出那些文件被以courier打头的进程打开，但是并不属于用户zahn lsof -p 30297 //显示那些文件被pid为30297的进程打开 lsof -D /tmp 显示所有在/tmp文件夹中打开的instance和文件的进程。但是symbol文件并不在列
lsof -u1000 //查看uid是100的用户的进程的文件使用情况 lsof -utony //查看用户tony的进程的文件使用情况 lsof -u^tony //查看不是用户tony的进程的文件使用情况(^是取反的意思) lsof -i //显示所有打开的端口 lsof -i:80 //显示所有打开80端口的进程 lsof -i -U //显示所有打开的端口和UNIX domain文件 lsof -i UDP@[url]www.akadia.com:123 //显示那些进程打开了到www.akadia.com的UDP的123(ntp)端口的链接 lsof -i tcp@ohaha.ks.edu.tw:ftp -r //不断查看目前ftp连接的情况(-r，lsof会永远不断的执行，直到收到中断信号,+r，lsof会一直执行，直到没有档案被显示,缺省是15s刷新) lsof -i tcp@ohaha.ks.edu.tw:ftp -n //lsof -n 不将IP转换为hostname，缺省是不加上-n参数

