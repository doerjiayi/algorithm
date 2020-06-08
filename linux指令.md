
#linux 指令

##head tail
情景linux–一张图搞懂head -n和tail -n
https://blog.csdn.net/signjing/article/details/69357769

linux取出某几行 
一、2113从第3000行开始，显示52611000行。即显示3000~3999行4102
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

##tee
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

##tcp dump
https://www.cnblogs.com/f-ck-need-u/p/7064286.html

##shell之三大文本处理工具grep、sed及awk 
https://www.cnblogs.com/-zyj/p/5763303.html

grep：文本过滤器，如果仅仅是过滤文本，可使用grep，其效率要比其他的高很多；
sed：Stream EDitor，流编辑器，默认只处理模式空间，不处理原数据，如果你处理的数据是针对行进行处理的，可以使用sed；
awk：报告生成器，格式化以后显示。如果对处理的数据需要生成报告之类的信息，或者你处理的数据是按列进行处理的，最好使用awk。
 
###grep
 
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

##find
find grep 联合使用 过滤所有子目录、文件
find . -type f -name '*.log*' | xargs grep --color -n 'GetConnectIdentify'


##traceroute
https://www.cnblogs.com/ftl1012/p/traceroute.html
traceroute -n -m 5 -q 4 -w 3 www.baidu.com 
说明： -n 显示IP地址，不查主机名，  -m 设置跳数   
         -q 4每个网关发送4个数据包    -w 把对外发探测包的等待响应时间设置为3秒
         
         
##Linux wc命令详解 
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

##ss和netstat的区别
https://blog.csdn.net/weixin_42816196/article/details/86580834

1）netstat参数和使用
常用参数-anplt
-a 显示所有活动的连接以及本机侦听的TCP、UDP端口
-l 显示监听的server port
-n 直接使用IP地址，不通过域名服务器
-p 正在使用Socket的程序PID和程序名称
-r 显示路由表
-t 显示TCP传输协议的连线状况
-u 显示UDP传输协议的连线状况
-w 显示RAW传输协议的连线状况

[root@king ~]# netstat -anutlp | grep 80
tcp6       0      0 :::80                   :::*                    LISTEN      8218/httpd     
1
2
在Linux下，raw格式的数据通常可以通过/proc/net/dev获得。在Windows平台，netstat信息可以通过IP Helper API的GetTcpTable和GetUdpTable函数获得。

2）ss（socket statistics）参数和使用
常用参数和netstat类似，如-anp
-a显示所有的sockets
-l显示正在监听的
-n显示数字IP和端口，不通过域名服务器
-p显示使用socket的对应的程序
-t只显示TCP sockets
-u只显示UDP sockets
-4 -6 只显示v4或v6V版本的sockets
-s打印出统计信息。这个选项不解析从各种源获得的socket。对于解析/proc/net/top大量的sockets计数时很有效
-0 显示PACKET sockets
-w 只显示RAW sockets
-x只显示UNIX域sockets
-r尝试进行域名解析，地址/端口
[root@king ~]# ss -anutlp | grep 80
tcp    LISTEN     0      128      :::80                   :::*                   users:(("httpd",pid=8223,fd=4),("httpd",pid=8222,fd=4),("httpd",pid=8221,fd=4),("httpd",pid=8220,fd=4),("httpd",pid=8219,fd=4),("httpd",pid=8218,fd=4))

3）原理对比
ss比netstat快的主要原因是，netstat是遍历/proc下面每个PID目录，ss直接读/proc/net下面的统计信息。所以ss执行的时候消耗资源以及消耗的时间都比netstat少很多。
当服务器的socket连接数量非常大时（如上万个），无论是使用netstat命令还是直接cat /proc/net/tcp执行速度都会很慢，相比之下ss可以节省很多时间。ss快的秘诀在于，它利用了TCP协议栈中tcp_diag，这是一个用于分析统计的模块，可以获得Linux内核中的第一手信息。如果系统中没有tcp_diag，ss也可以正常运行，只是效率会变得稍微慢但仍然比netstat要快。
根据博客http://www.cnblogs.com/wangkangluo1/archive/2012/05/15/2500844.html的测试
同样的大量socket连接情况下，netstat和ss获取同样的统计数据时的耗时，ss明显比netstat少很多。我们也可以简单测试一下在少数socket情况下（基本无差别）：
time netstat -atn以及time ss -atn对比
[root@king ~]# time ss -aut
Netid State      Recv-Q Send-Q                                              Local Address:Port                                                               Peer Address:Port                
udp   UNCONN     0      0                                                               *:sunrpc                                                                        *:*                    
udp   UNCONN     0      0                                                               *:entrust-ash                                                                   *:*                    
udp   UNCONN     0      0                                                       127.0.0.1:323                                                                           *:*                    
udp   UNCONN     0      0                                                              :::sunrpc                                                                       :::*                    
udp   UNCONN     0      0                                                              :::entrust-ash                                                                  :::*                    
udp   UNCONN     0      0                                                             ::1:323                                                                          :::*                    
tcp   LISTEN     0      128                                                             *:sunrpc                                                                        *:*                    
tcp   LISTEN     0      128                                                             *:ssh                                                                           *:*                    
tcp   LISTEN     0      100                                                     127.0.0.1:smtp                                                                          *:*                    
tcp   ESTAB      0      52                                                 192.168.159.11:ssh                                                               192.168.159.1:59869                
tcp   LISTEN     0      128                                                            :::sunrpc                                                                       :::*                    
tcp   LISTEN     0      128                                                            :::http                                                                         :::*                    
tcp   LISTEN     0      128                                                            :::ssh                                                                          :::*                    
tcp   LISTEN     0      100                                                           ::1:smtp                                                                         :::*                    

real	0m0.007s
user	0m0.003s
sys	0m0.003s
[root@king ~]# time netstat -aut
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 0.0.0.0:sunrpc          0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:ssh             0.0.0.0:*               LISTEN     
tcp        0      0 localhost:smtp          0.0.0.0:*               LISTEN     

tcp        0     52 king:ssh                192.168.159.1:59869     ESTABLISHED
tcp6       0      0 [::]:sunrpc             [::]:*                  LISTEN     
tcp6       0      0 [::]:http               [::]:*                  LISTEN     
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN     
tcp6       0      0 localhost:smtp          [::]:*                  LISTEN     
udp        0      0 0.0.0.0:sunrpc          0.0.0.0:*                          
udp        0      0 0.0.0.0:entrust-ash     0.0.0.0:*                          
udp        0      0 localhost:323           0.0.0.0:*                          
udp6       0      0 [::]:sunrpc             [::]:*                             
udp6       0      0 [::]:entrust-ash        [::]:*                             
udp6       0      0 localhost:323           [::]:*                             

real	0m5.242s
user	0m0.015s
sys	0m0.012s 

netstat属于net-tools工具集，ss属于ipoute工具集。

##Linux下的21个ss命令使用示例详解
https://renwole.com/archives/814

简介：
Socket Statistics（ss）命令类似于netstat，它用于显示各种有用的网络套接字信息。
长时间看，已经注意到netstat这个命令程序已经过时了。从而代替netstat的是ss命令。一个全新的ss命令使用起来必定有些陌生，不过ss许多选项与netstat使用的选项类似，但我们还会看到一些差异。
ss命令是Linux CentOS 7中iproute软件包的一部分，默认已经安装。
一般来说，网络套接字是由IP地址，传输协议和端口来定义的。这种组合构成了双向连接的一个方面。例如：一个Web服务器可能正在侦听172.28.204.62:80上的传入TCP连接，这是套接字。不过需要说明的是套接字不是连接本身，而是连接的端点之一。
下面我讲解如何使用ss命令查看各种信息。具体使用语法如下：
ss [options] [ FILTER ]
###1.列出已建立的连接
默认情况下，如果我们运行ss命令而没有指定其他选项，它将显示所有已建立连接的打开的非侦听套接字的列表，例如TCP，UDP或UNIX套接字。
[root@renwolecom ~]# ss | head -n 5
Netid  State  Recv-Q Send-Q Local Address:Port   Peer Address:Port
u_str  ESTAB  0      0       * 19098                 * 18222
u_str  ESTAB  0      0       * 19441                 * 19440
u_str  ESTAB  0      0       * 19440                 * 19441
u_str  ESTAB  0      0       * 19396                 * 19397

###2.显示监听套接字
我们可以使用-l选项专门列出当前正在侦听连接的套接字，而不是列出所有的套接字。
[root@renwolecom ~]# ss -lt
State   Recv-Q Send-Q    Local Address:Port       Peer Address:Port
LISTEN  0      128                   *:http                  *:*
LISTEN  0      100           127.0.0.1:smtp                  *:*
LISTEN  0      128                   *:entexthigh            *:*
LISTEN  0      128       172.28.204.62:zabbix-trapper        *:*
LISTEN  0      128           127.0.0.1:cslistener            *:*
LISTEN  0      80                   :::mysql                :::*
LISTEN  0      100                 ::1:smtp                 :::*
LISTEN  0      128                  :::entexthigh           :::*
在这个示例中，我们还使用-t选项只列出TCP，稍后将对此进行详细说明。在后面的例子中，你会看到我将结合多种选择，以快速过滤掉，从而达到我们的目的。
###3.显示进程
我们可以用-p选项打印出拥有套接字的进程或PID号。
[root@renwolecom ~]# ss -pl

Netid  State      Recv-Q Send-Q Local Address:Port     Peer Address:Port
tcp    LISTEN     0      128    :::http                :::*                 users:(("httpd",pid=10522,fd=4),("httpd",pid=10521,fd=4),("httpd",pid=10520,fd=4),("httpd",pid=10519,fd=4),("httpd",pid=10518,fd=4),("httpd",pid=10516,fd=4))
在上面的例子中我只列出了一个结果，没有进行进一步选项，因为ss的完整输出打印出超过500行到标准输出。所以我只列出一条结果，由此我们可以看到服务器上运行的各种Apache进程ID。
###4.不解析服务名称
默认情况下，ss只会解析端口号，例如在下面的行中，我们可以看到172.28.204.62:mysql，其中mysql被列为本地端口。
[root@renwolecom ~]# ss
Netid State Recv-Q Send-Q   Local Address:Port      	Peer Address:Port
tcp   ESTAB 0      0 ::ffff:172.28.204.62:mysql ::ffff:172.28.204.62:38920
tcp   ESTAB 0      0 ::ffff:172.28.204.62:mysql ::ffff:172.28.204.62:51598
tcp   ESTAB 0      0 ::ffff:172.28.204.62:mysql ::ffff:172.28.204.62:51434
tcp   ESTAB 0      0 ::ffff:172.28.204.62:mysql ::ffff:172.28.204.62:36360
但是，如果我们指定-n选项，看到的是端口号而不是服务名称。
[root@renwolecom ~]# ss -n
Netid State Recv-Q Send-Q   Local Address:Port      	Peer Address:Port
tcp   ESTAB 0      0 ::ffff:172.28.204.62:3306  ::ffff:172.28.204.62:38920
tcp   ESTAB 0      0 ::ffff:172.28.204.62:3306  ::ffff:172.28.204.62:51598
tcp   ESTAB 0      0 ::ffff:172.28.204.62:3306  ::ffff:172.28.204.62:51434
tcp   ESTAB 0      0 ::ffff:172.28.204.62:3306  ::ffff:172.28.204.62:36360
现在显示3306，而非mysql，因为禁用了主机名和端口的所有名称解析。另外你还可以查看/etc/services得到所有服务对应的端口列表。
###5.解析数字地址/端口
用-r选项可以解析IP地址和端口号。用此方法可以列出172.28.204.62服务器的主机名。
[root@renwolecom ~]# ss -r
Netid  State  Recv-Q Send-Q        Local Address:Port      Peer Address:Port
tcp    ESTAB      0      0         renwolecom:mysql        renwolecom:48134
###6.IPv4套接字
我们可以通过-4选项只显示与IPv4套接字对应的信息。在下面的例子中，我们还使用-l选项列出了在IPv4地址上监听的所有内容。
[root@renwolecom ~]# ss -l4
Netid State      Recv-Q Send-Q  Local Address:Port            Peer Address:Port
tcp   LISTEN     0      128                 *:http                       *:*
tcp   LISTEN     0      100         127.0.0.1:smtp                       *:*
tcp   LISTEN     0      128                 *:entexthigh                 *:*
tcp   LISTEN     0      128     172.28.204.62:zabbix-trapper             *:*
tcp   LISTEN     0      128         127.0.0.1:cslistener                 *:*
###7.IPv6套接字
同样，我们可以使用-6选项只显示与IPv6套接字相关信息。在下面的例子中，我们还使用-l选项列出了在IPv6地址上监听的所有内容。
[root@renwolecom ~]# ss -l6
Netid State      Recv-Q Send-Q  Local Address:Port            Peer Address:Port
udp   UNCONN     0      0                  :::ipv6-icmp                 :::*
udp   UNCONN     0      0                  :::ipv6-icmp                 :::*
udp   UNCONN     0      0                  :::21581                     :::*
tcp   LISTEN     0      80                 :::mysql                     :::*
tcp   LISTEN     0      100               ::1:smtp                      :::*
tcp   LISTEN     0      128                :::entexthigh                :::*
###8.只显示TCP
-t选项只显示TCP套接字。当与-l结合只打印出监听套接字时，我们可以看到所有在TCP上侦听的内容。
[root@renwolecom ~]# ss -lt
State       Recv-Q Send-Q    Local Address:Port               Peer Address:Port
LISTEN      0      128                   *:http                          *:*
LISTEN      0      100           127.0.0.1:smtp                          *:*
LISTEN      0      128                   *:entexthigh                    *:*
LISTEN      0      128       172.28.204.62:zabbix-trapper                *:*
LISTEN      0      128           127.0.0.1:cslistener                    *:*
LISTEN      0      80                   :::mysql                        :::*
LISTEN      0      100                 ::1:smtp                         :::*
LISTEN      0      128                  :::entexthigh                   :::*
###9.显示UDP
-u选项可用于仅显示UDP套接字。由于UDP是一种无连接的协议，因此只运行-u选项将不显示输出，我们可以将它与-a或-l选项结合使用，来查看所有侦听UDP套接字，如下所示：
[root@renwolecom ~]# ss -ul
State      Recv-Q Send-Q Local Address:Port                 Peer Address:Port
UNCONN     0      0              *:sunwebadmins                 *:*
UNCONN     0      0              *:etlservicemgr                *:*
UNCONN     0      0              *:dynamid                      *:*
UNCONN     0      0              *:9003                         *:*
UNCONN     0      0              *:9004                         *:*
UNCONN     0      0      127.0.0.1:terabase                     *:*
UNCONN     0      0              *:56803                        *:*
###10. Unix套接字
-x选项只能用来显示unix域套接字。
[root@renwolecom ~]# ss -x
Netid State Recv-Q Send-Q                    Local Address:Port Peer Address:Port
u_str ESTAB 0      0 /tmp/zabbix_server_preprocessing.sock 23555           * 21093
u_str ESTAB 0      0          /tmp/zabbix_server_ipmi.sock 20155           * 19009
u_str ESTAB 0      0 /tmp/zabbix_server_preprocessing.sock 19354           * 22573
u_str ESTAB 0      0 /tmp/zabbix_server_preprocessing.sock 21844           * 19375
...
###11.显示所有信息
-a选项显示所有的监听和非监听套接字，在TCP的情况下，这意味着已建立的连接。这个选项与其他的组合很有用，例如可以添加-a选项显示所有的UDP套接字，默认情况下只有-u选项我们看不到多少信息。
[root@renwolecom ~]# ss -u
Recv-Q Send-Q Local Address:Port                 Peer Address:Port
0      0      172.28.204.66:36371                    8.8.8.8:domain
[root@renwolecom ~]# ss -ua
State      Recv-Q Send-Q Local Address:Port                 Peer Address:Port
UNCONN     0      0                 *:sunwebadmins                    *:*
UNCONN     0      0                 *:etlservicemgr                   *:*
UNCONN     0      0                 *:dynamid                         *:*
UNCONN     0      0                 *:9003                            *:*
UNCONN     0      0                 *:9004                            *:*
UNCONN     0      0         127.0.0.1:terabase                        *:*
UNCONN     0      0                 *:56803                           *:*
ESTAB      0      0      172.28.204.66:36371                     8.8.8.8:domain
###12.显示套接字内存使用情况
-m选项可用于显示每个套接字使用的内存量。
[root@renwolecom ~]# ss -ltm
State   Recv-Q Send-Q  Local Address:Port Peer Address:Port
LISTEN  0      128                 *:http           *:*skmem:(r0,rb87380,t0,tb16384,f0,w0,o0,bl0)
LISTEN  0      100         127.0.0.1:smtp           *:*skmem:(r0,rb87380,t0,tb16384,f0,w0,o0,bl0)
LISTEN  0      128                 *:entexthigh     *:*skmem:(r0,rb87380,t0,tb16384,f0,w0,o0,bl0)
LISTEN  0      128     172.28.204.62:zabbix-trapper *:*skmem:(r0,rb87380,t0,tb16384,f0,w0,o0,bl0)
LISTEN  0      128         127.0.0.1:cslistener     *:*skmem:(r0,rb87380,t0,tb16384,f0,w0,o0,bl0)
LISTEN  0      80                 :::mysql         :::*skmem:(r0,rb87380,t0,tb16384,f0,w0,o0,bl0)
LISTEN  0      100               ::1:smtp          :::*skmem:(r0,rb87380,t0,tb16384,f0,w0,o0,bl0)
LISTEN  0      128                :::entexthigh    :::*skmem:(r0,rb87380,t0,tb16384,f0,w0,o0,bl0)
###13.显示TCP内部信息
我们可以使用-i选项请求额外的内部TCP信息。
[root@renwolecom ~]# ss -lti
State       Recv-Q Send-Q Local Address:Port         Peer Address:Port
LISTEN      0      128               *:chimera-hwm              *:* 	bbr cwnd:10
LISTEN      0      128               *:etlservicemgr            *:* 	bbr cwnd:10
LISTEN      0      128   172.28.204.66:27017                    *:* 	bbr cwnd:10
LISTEN      0      128       127.0.0.1:27017                    *:* 	bbr cwnd:10
LISTEN      0      128               *:dynamid                  *:* 	bbr cwnd:10
LISTEN      0      128               *:9003                     *:* 	bbr cwnd:10
LISTEN      0      128               *:9004                     *:* 	bbr cwnd:10
LISTEN      0      128               *:http                     *:* 	bbr cwnd:10
LISTEN      0      128               *:ssh                      *:* 	bbr cwnd:10
LISTEN      0      100       127.0.0.1:smtp                     *:* 	bbr cwnd:10
LISTEN      0      128               *:sunwebadmins             *:* 	bbr cwnd:10
LISTEN      0      128              :::ssh                     :::* 	bbr cwnd:10
在每个侦听套接字下面，我们可以看到更多信息。注意：-i选项不适用于UDP，如果您指定-u，而非-t，则不会显示这些额外的信息。
###14.显示统计信息
我们可以使用-s选项快速查看统计数据。
[root@renwolecom ~]# ss -s
Total: 798 (kernel 1122)
TCP:   192 (estab 99, closed 81, orphaned 0, synrecv 0, timewait 1/0), ports 0

Transport Total     IP        IPv6
*         1122      -         -
RAW       1         0         1
UDP       0         0         0
TCP       111       59        52
INET      112       59        53
FRAG      0         0         0
这使我们能够快速看到已建立连接的总数，及各种类型的套接字的计数和IPv4或IPv6的使用情况。

###15.基于状态的过滤器
我们可以指定一个套接字的状态，只打印这个状态下的套接字。例如，我们可以指定包括已建立， established, syn-sent, syn-recv, fin-wait-1, fin-wait-2, time-wait, closed, closed-wait, last-ack监听和关闭等状态。以下示例显示了所有建立的TCP连接。为了生成这个，我通过SSH连接到了服务器，并从Apache加载了一个网页。然后我们可以看到与Apache的连接迅速转变为等待时间。
[root@renwolecom ~]# ss -t state established
Recv-Q Send-Q       Local Address:Port              Peer Address:Port
0      52           172.28.204.67:ssh              123.125.71.38:49518
0      0     ::ffff:172.28.204.67:http      ::ffff:123.125.71.38:49237
[root@renwolecom ~]# ss -t state established
Recv-Q Send-Q       Local Address:Port              Peer Address:Port
0      0            172.28.204.67:ssh            103.240.143.126:55682
0      52           172.28.204.67:ssh              123.125.71.38:49518
0      0     ::ffff:172.28.204.67:http      ::ffff:123.125.71.38:49262

###16.根据端口号进行过滤
可以通过过滤还可以列出小于(lt)，大于(gt)，等于(eq)，不等于(ne)，小于或等于(le)，或大于或等于(ge)的所有端口。
例如，以下命令显示端口号为500或以下的所有侦听端口：
[root@renwolecom ~]# ss -ltn sport le 500
State       Recv-Q Send-Q Local Address:Port        Peer Address:Port
LISTEN      0      128                *:80                     *:*
LISTEN      0      100        127.0.0.1:25                     *:*
LISTEN      0      100              ::1:25                    :::*
为了进行比较，我们可以执行相反的操作，并查看大于500的所有端口：
[root@renwolecom ~]# ss -ltn sport gt 500
State       Recv-Q Send-Q Local Address:Port        Peer Address:Port
LISTEN      0      128                *:12002                  *:*
LISTEN      0      128    172.28.204.62:10051                  *:*
LISTEN      0      128        127.0.0.1:9000                   *:*
LISTEN      0      80                :::3306                  :::*
LISTEN      0      128               :::12002                 :::*
我们还可以根据源或目标端口等项进行筛选，例如，我们搜索具有SSH源端口运行的TCP套接字：
[root@renwolecom ~]# ss -t '( sport = :ssh )'
State       Recv-Q Send-Q    Local Address:Port     Peer Address:Port
ESTAB       0      0         172.28.204.66:ssh     123.125.71.38:50140
###17.显示SELinux上下文
-Z与-z选项可用于显示套接字的SELinux安全上下文。 在下面的例子中，我们使用-t和-l选项来列出侦听的TCP套接字，使用-Z选项我们也可以看到SELinux的上下文。
[root@renwolecom ~]# ss -tlZ
State  Recv-Q Send-Q  Local Address:Port        Peer Address:Port
LISTEN 0      128                 *:sunrpc                 *:*
users:(("systemd",pid=1,proc_ctx=system_u:system_r:init_t:s0,fd=71))
LISTEN 0      5       172.28.204.62:domain                 *:*
users:(("dnsmasq",pid=1810,proc_ctx=system_u:system_r:dnsmasq_t:s0-s0:c0.c1023,fd=6))
LISTEN 0      128                 *:ssh                    *:*
users:(("sshd",pid=1173,proc_ctx=system_u:system_r:sshd_t:s0-s0:c0.c1023,fd=3))
LISTEN 0      128         127.0.0.1:ipp                    *:*
users:(("cupsd",pid=1145,proc_ctx=system_u:system_r:cupsd_t:s0-s0:c0.c1023,fd=12))
LISTEN 0      100         127.0.0.1:smtp                   *:*
users:(("master",pid=1752,proc_ctx=system_u:system_r:postfix_master_t:s0,fd=13))
###18.显示版本号
-v选项可用于显示ss命令的特定版本信息，在这种情况下，我们可以看到提供ss的iproute包的版本。
[root@renwolecom ~]# ss -v
ss utility, iproute2-ss130716
###19.显示帮助文档信息
-h选项可用于显示有关ss命令的进一步的帮助，如果需要对最常用的一些选项进行简短说明，则可以将其用作快速参考。 请注意：这里并未输入完整列表。
[root@renwolecom ~]# ss -h
Usage: ss [ OPTIONS ]
###20.显示扩展信息
我们可以使用-e选项来显示扩展的详细信息，如下所示，我们可以看到附加到每条行尾的扩展信息。
[root@renwolecom ~]# ss -lte
State  Recv-Q Send-Q Local Address:Port   Peer Address:Port
LISTEN 0      128                *:sunrpc *:*      ino:16090 sk:ffff880000100000 <->
LISTEN 0      5      172.28.204.62:domain *:*      ino:23750 sk:ffff880073e70f80 <->
LISTEN 0      128                *:ssh    *:*      ino:22789 sk:ffff880073e70000 <->
LISTEN 0      128        127.0.0.1:ipp    *:*      ino:23091 sk:ffff880073e707c0 <->
LISTEN 0      100        127.0.0.1:smtp   *:*      ino:24659 sk:ffff880000100f80 <->
###21.显示计时器信息
-o选项可用于显示计时器信息。该信息向我们展示了诸如重新传输计时器值、已经发生的重新传输的数量以及已发送的keepalive探测的数量。
[root@renwolecom ~]# ss -to
State      Recv-Q Send-Q Local Address:Port      Peer Address:Port
ESTAB      0      52     172.28.204.67:ssh      123.125.71.38:49518timer:(on,406ms,0)
LAST-ACK   0      1      172.28.204.67:ssh    103.240.143.126:49603timer:(on,246ms,0)
总结：
现在你应该对ss有了初步的认识。如果你想使用ss命令快速检查有关套接字的各种信息，建议你查阅ss的相关手册。
