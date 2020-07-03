
#TCP/IP四层模型讲解笔记
https://www.cnblogs.com/ssooking/p/6323483.html

##TCP协议包
https://blog.csdn.net/u014590757/article/details/79901948
TCP首部格式

　tcp数据是被封装在IP数据包中的，和udp类似，在IP数据包的数据部分。tcp数据包的格式如下： 
源端口号和目的端口号（寻址）与udp中类似，用于寻找发端和收端应用进程。这两个值加上IP首部中的源端IP地址和目的端IP地址唯一确定一个，在网络编程中，一般一个IP地址和一个端口号组合称为一个套接字（socket）。 
　　序号（seq）：用来标识从TCP发端向TCP收端发送的数据字节流，它表示在这个报文段中的第一个数据字节。在tcp中tcp用序号对每个字节进行计数（这个值与发送的帧数没有关系，而是与发送的数据字节数有关系，后面会有说明）。 
确认序号（seq+1）：包含发送确认的一端所期望收到的下一个序号。因此，确认序号应当是上次已成功收到数据字节序号加 1（不是单纯的序号加1，还包括数据字节数）。 

首部长度：用于记录tcp数据报首部的长度，一般为20字节，实际值为首部长度除以4。 

下面的是标志位对应的功能：
URG： 紧急指针（ urgent pointer）有效。 
ACK： 确认序号有效。 
PSH： 接收方应该尽快将这个报文段交给应用层。 
RST： 重建连接。 
SYN： 同步序号用来发起一个连接。 
FIN： 发端完成发送任务。 
窗口大小：用于流量控制。 
检验和：检验和覆盖了整个的 TCP报文段： TCP首部和TCP数据，与udp相似需要计算伪首部。


（1）每个TCP段都包括源端和目的端的端口号，用于寻找发送端和接收端的应用进程。这两个值加上IP首部的源端IP地址和目的端IP地址唯一确定一个TCP连接。
（2）序号用来标识从TCP发送端向接收端发送的数据字节流，它表示在这个报文段中的第一个数据字节。如果将字节流看作在两个应用程序间的单向流动，则TCP用序号对每个字节进行计数(seq)。
（3）当建立一个新连接时，SYN标志变1。序号字段包含由这个主机选择的该连接的初始序号ISN，该主机要发送数据的第一个字节的序号为这个ISN加1，因为SYN标志使用了一个序号。
（4）既然每个被传输的字节都被计数，确认序号包含发送确认的一端所期望收到的下一个序号。因此，确认序号应当时上次已成功收到数据字节序号加1。只有ACK标志为1时确认序号字段才有效。(ack)
（5）发送ACK无需任何代价，因为32位的确认序号字段和ACK标志一样，总是TCP首部的一部分。因此一旦一个连接建立起来，这个字段总是被设置，ACK标志也总是被设置为1。
（6）TCP为应用层提供全双工的服务。因此，连接的每一端必须保持每个方向上的传输数据序号。
（7）TCP可以表述为一个没有选择确认或否认的滑动窗口协议。因此TCP首部中的确认序号表示发送方已成功收到字节，但还不包含确认序号所指的字节。当前还无法对数据流中选定的部分进行确认。
（8）首部长度需要设置，因为任选字段的长度是可变的。TCP首部最多60个字节。
（9）6个标志位中的多个可同时设置为1
◆URG－紧急指针有效
◆ ACK－确认序号有效
◆ PSH－接收方应尽快将这个报文段交给应用层
◆ RST－重建连接
◆ SYN－同步序号用来发起一个连接
◆ FIN－发送端完成发送任务
（10）TCP的流量控制由连接的每一端通过声明的窗口大小来提供。窗口大小为字节数，起始于确认序号字段指明的值，这个值是接收端期望接收的字节数。窗口大小是一个16为的字段，因而窗口大小最大为65535字节。
（11）检验和覆盖整个TCP报文端：TCP首部和TCP数据。这是一个强制性的字段，一定是由发送端计算和存储，并由接收端进行验证。TCP检验和的计算和UDP首部检验和的计算一样，也使用伪首部。
（12）紧急指针是一个正的偏移量，序号字段中的值相加表示紧急数据最后一个字节的序号。TCP的紧急方式是发送端向另一端发送紧急数据的一种方式。

（13）最常见的可选字段是最长报文大小MMS，每个连接方通常都在通信的第一个报文段中指明这个选项。它指明本端所能接收的最大长度的报文段。 


　TCP是一种面向连接的、可靠的、基于字节流的传输层通信协议。 
　　面向连接： 面向连接意味着使用tcp的应用程序在传输数据前必须先建立连接，就如打电话一样，要先进行拨号，等待对方响应才能开始说话。 
可靠性：tcp协议通过下列方式来提高可靠性： 
·        1.应用数据被分割成TCP认为最适合发送的数据块。这和UDP完全不同，应用程序产生的数据报长度将保持不变。由TCP传递给IP的信息单位称为报文段或段。（定长）
·        2.当TCP发出一个段后，它启动一个定时器，等待目的端确认收到这个报文段。如果不能及时收到一个确认，将重发这个报文段。（定时重发）
·        3.当TCP收到发自TCP连接另一端的数据，它将发送一个确认。这个确认不是立即发送，通常将推迟几分之一秒。（确认机制）
·        4.TCP将保持它首部和数据的检验和。这是一个端到端的检验和，目的是检测数据在传输过程中的任何变化。如果收到段的检验和有差错，TCP将丢弃这个报文段和不确认收到此报文段（希望发端超时并重发）。（检验和较检）
·        5.既然TCP报文段作为IP数据报来传输，而IP数据报的到达可能会失序，因此TCP报文段的到达也可能会失序。如果必要，TCP将对收到的数据进行重新排序，将收到的数据以正确的顺序交给应用层。（必要时重新排序）
·        6.既然I P数据报会发生重复，TCP的接收端必须丢弃重复的数据。（去重）
·        7.TCP还能提供流量控制。TCP连接的每一方都有固定大小的缓冲空间。TCP的接收端只允许另一端发送接收端缓冲区所能接纳的数据。这将防止较快主机致使较慢主机的缓冲区溢出。（流量控制）
字节流：两个应用程序通过TCP连接交换8 bit字节构成的字节流。
　　另外，TCP对字节流的内容不作任何解释。TCP不知道传输的数据字节流是二进制数据，还是ASCII字符或者其他类型数据。对字节流的解释由TCP连接双方的应用层解释。


##TCP拥塞控制，抓包
https://blog.csdn.net/weixin_45237113/article/details/92799530

1、默认启动
1 tcpdump -vv
普通情况下，直接启动tcpdump将监视第一个网络接口上所有流过的数据包。

2、过滤主机
1 tcpdump -i eth1 host 192.168.1.1
2 tcpdump -i eth1 src host 192.168.1.1
3 tcpdump -i eth1 dst host 192.168.1.1
抓取所有经过eth1，目的或源地址是192.168.1.1的网络数据

指定源地址，192.168.1.1

指定目的地址，192.168.1.1

3、过滤端口
1 tcpdump -i eth1 port 25
2 tcpdump -i eth1 src port 25
3 tcpdump -i eth1 dst port 25
抓取所有经过eth1，目的或源端口是25的网络数据

指定源端口

指定目的端口

4、网络过滤
1 tcpdump -i eth1 net 192.168
2 tcpdump -i eth1 src net 192.168
3 tcpdump -i eth1 dst net 192.168
5、协议过滤
1 tcpdump -i eth1 arp
2 tcpdump -i eth1 ip
3 tcpdump -i eth1 tcp
4 tcpdump -i eth1 udp
5 tcpdump -i eth1 icmp
6、常用表达式
1 非 : ! or “not” (去掉双引号)
2 且 : && or “and”
3 或 : || or “or”
抓取所有经过eth1，目的地址是192.168.1.254或192.168.1.200端口是80的TCP数

1 tcpdump -i eth1 ‘((tcp) and (port 80) and ((dst host 192.168.1.254) or (dst host 192.168.1.200)))’
抓取所有经过eth1，目标MAC地址是00:01:02:03:04:05的ICMP数据

1 tcpdump -i eth1 ‘((icmp) and ((ether dst host 00:01:02:03:04:05)))’
抓取所有经过eth1，目的网络是192.168，但目的主机不是192.168.1.200的TCP数据

1 tcpdump -i eth1 ‘((tcp) and ((dst net 192.168) and (not dst host 192.168.1.200)))’


##理解tcp流的相关属性，
如何识别TCP流
给你一个报文包,求问如何识别出一个TCP流
用五元组,(源ip,目的ip,源端口,目的端口,协议号(TCP)),根据这个就能识别一个tcp流了

##tcp连接出现问题的排查方法
https://blog.csdn.net/hguisu/article/details/38700899

我们通过了解TCP各个状态，可以排除和定位网络或系统故障时大有帮助。（总结网络上的内容）
 
1、TCP状态
了解TCP之前，先了解几个命令：

linux查看tcp的状态命令：
1）、netstat -nat  查看TCP各个状态的数量
2）、lsof  -i:port  可以检测到打开套接字的状况
3)、  sar -n SOCK 查看tcp创建的连接数
4)、tcpdump -iany tcp port 9000 对tcp端口为9000的进行抓包
5)、tcpdump   dst  port 9000 -w dump9000.pcap  对tcp目标端口为9000的进行抓包保存pcap文件wireshark分析。
6)、tcpdump tcp port 9000  -n -X -s 0 -w tcp.cap 对tcp/http目标端口为9000的进行抓包保存pcap文件wireshark分析。

网络测试常用命令; 
1）ping:检测网络连接的正常与否,主要是测试延时、抖动、丢包率。

     但是很多服务器为了防止攻击，一般会关闭对ping的响应。所以ping一般作为测试连通性使用。ping命令后，会接收到对方发送的回馈信息，其中记录着对方的IP地址和TTL。TTL是该字段指定IP包被路由器丢弃之前允许通过的最大网段数量。TTL是IPv4包头的一个8 bit字段。例如IP包在服务器中发送前设置的TTL是64，你使用ping命令后，得到服务器反馈的信息，其中的TTL为56，说明途中一共经过了8道路由器的转发，每经过一个路由，TTL减1。

2）traceroute：raceroute 跟踪数据包到达网络主机所经过的路由工具

   traceroute hostname

3）pathping：是一个路由跟踪工具，它将 ping 和 tracert 命令的功能与这两个工具所不提供的其他信息结合起来，综合了二者的功能

   pathping www.baidu.com

4）mtr：以结合ping nslookup tracert 来判断网络的相关特性

5) nslookup:用于解析域名，一般用来检测本机的DNS设置是否配置正确。

LISTENING：侦听来自远方的TCP端口的连接请求.

   首先服务端需要打开一个socket进行监听，状态为LISTEN。

    有提供某种服务才会处于LISTENING状态，TCP状态变化就是某个端口的状态变化，提供一个服务就打开一个端口，例如：提供www服务默认开的是80端口，提供ftp服务默认的端口为21，当提供的服务没有被连接时就处于LISTENING状态。FTP服务启动后首先处于侦听（LISTENING）状态。处于侦听LISTENING状态时，该端口是开放的，等待连接，但还没有被连接。就像你房子的门已经敞开的，但还没有人进来。
    看LISTENING状态最主要的是看本机开了哪些端口，这些端口都是哪个程序开的，关闭不必要的端口是保证安全的一个非常重要的方面，服务端口都对应一个服务（应用程序），停止该服务就关闭了该端口，例如要关闭21端口只要停止IIS服务中的FTP服务即可。关于这方面的知识请参阅其它文章。
    如果你不幸中了服务端口的木马，木马也开个端口处于LISTENING状态。

SYN-SENT：客户端SYN_SENT状态：

         再发送连接请求后等待匹配的连接请求:客户端通过应用程序调用connect进行active open.于是客户端tcp发送一个SYN以请求建立一个连接.之后状态置为SYN_SENT. /*The socket is actively attempting to establish a connection. 在发送连接请求后等待匹配的连接请求 */
     当请求连接时客户端首先要发送同步信号给要访问的机器，此时状态为SYN_SENT，如果连接成功了就变为ESTABLISHED，正常情况下SYN_SENT状态非常短暂。例如要访问网站http://www.baidu.com,如果是正常连接的话，用TCPView观察IEXPLORE.EXE（IE）建立的连接会发现很快从SYN_SENT变为ESTABLISHED，表示连接成功。SYN_SENT状态快的也许看不到。
     如果发现有很多SYN_SENT出现，那一般有这么几种情况，一是你要访问的网站不存在或线路不好，二是用扫描软件扫描一个网段的机器，也会出出现很多SYN_SENT，另外就是可能中了病毒了，例如中了"冲击波"，病毒发作时会扫描其它机器，这样会有很多SYN_SENT出现。


SYN-RECEIVED：服务器端状态SYN_RCVD
       再收到和发送一个连接请求后等待对方对连接请求的确认

       当服务器收到客户端发送的同步信号时，将标志位ACK和SYN置1发送给客户端，此时服务器端处于SYN_RCVD状态，如果连接成功了就变为ESTABLISHED，正常情况下SYN_RCVD状态非常短暂。
   如果发现有很多SYN_RCVD状态，那你的机器有可能被SYN Flood的DoS(拒绝服务攻击)攻击了。

   SYN Flood的攻击原理是：

      在进行三次握手时，攻击软件向被攻击的服务器发送SYN连接请求（握手的第一步），但是这个地址是伪造的，如攻击软件随机伪造了51.133.163.104、65.158.99.152等等地址。服务器在收到连接请求时将标志位ACK和SYN置1发送给客户端（握手的第二步），但是这些客户端的IP地址都是伪造的，服务器根本找不到客户机，也就是说握手的第三步不可能完成。

       这种情况下服务器端一般会重试（再次发送SYN+ACK给客户端）并等待一段时间后丢弃这个未完成的连接，这段时间的长度我们称为SYN Timeout，一般来说这个时间是分钟的数量级（大约为30秒-2分钟）；一个用户出现异常导致服务器的一个线程等待1分钟并不是什么很大的问题，但如果有一个恶意的攻击者大量模拟这种情况，服务器端将为了维护一个非常大的半连接列表而消耗非常多的资源----数以万计的半连接，即使是简单的保存并遍历也会消耗非常多的CPU时间和内存，何况还要不断对这个列表中的IP进行SYN+ACK的重试。此时从正常客户的角度看来，服务器失去响应，这种情况我们称做：服务器端受到了SYN Flood攻击（SYN洪水攻击）



ESTABLISHED：代表一个打开的连接。
     ESTABLISHED状态是表示两台机器正在传输数据，观察这个状态最主要的就是看哪个程序正在处于ESTABLISHED状态。
     服务器出现很多ESTABLISHED状态： netstat -nat |grep 9502或者使用lsof  -i:9502可以检测到。
       当客户端未主动close的时候就断开连接：即客户端发送的FIN丢失或未发送。
        这时候若客户端断开的时候发送了FIN包，则服务端将会处于CLOSE_WAIT状态；
         这时候若客户端断开的时候未发送FIN包，则服务端处还是显示ESTABLISHED状态；
          结果客户端重新连接服务器。

          而新连接上来的客户端（也就是刚才断掉的重新连上来了）在服务端肯定是ESTABLISHED; 如果客户端重复的上演这种情况，那么服务端将会出现大量的假的ESTABLISHED连接和CLOSE_WAIT连接。

         最终结果就是新的其他客户端无法连接上来，但是利用netstat还是能看到一条连接已经建立，并显示ESTABLISHED，但始终无法进入程序代码。


FIN-WAIT-1：等待远程TCP连接中断请求，或先前的连接中断请求的确认

       主动关闭(active close)端应用程序调用close，于是其TCP发出FIN请求主动关闭连接，之后进入FIN_WAIT1状态./* The socket is closed, and the connection is shutting down. 等待远程TCP的连接中断请求，或先前的连接中断请求的确认 */

      如果服务器出现shutdown再重启，使用netstat -nat查看，就会看到很多FIN-WAIT-1的状态。就是因为服务器当前有很多客户端连接，直接关闭服务器后，无法接收到客户端的ACK。

FIN-WAIT-2：从远程TCP等待连接中断请求

       主动关闭端接到ACK后，就进入了FIN-WAIT-2 ./* Connection is closed, and the socket is waiting for a shutdown from the remote end. 从远程TCP等待连接中断请求 */

        这就是著名的半关闭的状态了，这是在关闭连接时，客户端和服务器两次握手之后的状态。在这个状态下，应用程序还有接受数据的能力，但是已经无法发送数据，但是也有一种可能是，客户端一直处于FIN_WAIT_2状态，而服务器则一直处于WAIT_CLOSE状态，而直到应用层来决定关闭这个状态。


CLOSE-WAIT：等待从本地用户发来的连接中断请求

         被动关闭(passive close)端TCP接到FIN后，就发出ACK以回应FIN请求(它的接收也作为文件结束符传递给上层应用程序),并进入CLOSE_WAIT. /* The remote end has shut down, waiting for the socket to close. 等待从本地用户发来的连接中断请求 */

       

CLOSING：等待远程TCP对连接中断的确认

比较少见./* Both sockets are shut down but we still don't have all our data sent. 等待远程TCP对连接中断的确认 */

LAST-ACK：等待原来的发向远程TCP的连接中断请求的确认

被动关闭端一段时间后，接收到文件结束符的应用程序将调用CLOSE关闭连接。这导致它的TCP也发送一个 FIN,等待对方的ACK.就进入了LAST-ACK . /* The remote end has shut down, and the socket is closed. Waiting for acknowledgement. 等待原来发向远程TCP的连接中断请求的确认 */

使用并发压力测试的时候，突然断开压力测试客户端，服务器会看到很多LAST-ACK。


TIME-WAIT：等待足够的时间以确保远程TCP接收到连接中断请求的确认

在主动关闭端接收到FIN后，TCP就发送ACK包，并进入TIME-WAIT状态。/* The socket is waiting after close to handle packets still in the network.等待足够的时间以确保远程TCP接收到连接中断请求的确认 */

            TIME_WAIT等待状态，这个状态又叫做2MSL状态，说的是在TIME_WAIT2发送了最后一个ACK数据报以后，要进入TIME_WAIT状态，这个状态是防止最后一次握手的数据报没有传送到对方那里而准备的（注意这不是四次握手，这是第四次握手的保险状态）。这个状态在很大程度上保证了双方都可以正常结束，但是，问题也来了。

由于插口的2MSL状态（插口是IP和端口对的意思，socket），使得应用程序在2MSL时间内是无法再次使用同一个插口的，对于客户程序还好一些，但是对于服务程序，例如httpd，它总是要使用同一个端口来进行服务，而在2MSL时间内，启动httpd就会出现错误（插口被使用）。为了避免这个错误，服务器给出了一个平静时间的概念，这是说在2MSL时间内，虽然可以重新启动服务器，但是这个服务器还是要平静的等待2MSL时间的过去才能进行下一次连接。

           详情请看：TIME_WAIT引起Cannot assign requested address报错

CLOSED：没有任何连接状态

被动关闭端在接受到ACK包后，就进入了closed的状态。连接结束./* The socket is not being used. 没有任何连接状态 */

客户端应用程序的状态迁移图

        客户端的状态可以用如下的流程来表示：

        CLOSED->SYN_SENT->ESTABLISHED->FIN_WAIT_1->FIN_WAIT_2->TIME_WAIT->CLOSED

         以上流程是在程序正常的情况下应该有的流程，从书中的图中可以看到，在建立连接时，当客户端收到SYN报文的ACK以后，客户端就打开了数据交互地连接。而结束连接则通常是客户端主动结束的，客户端结束应用程序以后，需要经历FIN_WAIT_1，FIN_WAIT_2等状态，这些状态的迁移就是前面提到的结束连接的四次握手。

服务器的状态迁移图

        服务器的状态可以用如下的流程来表示：

         CLOSED->LISTEN->SYN收到->ESTABLISHED->CLOSE_WAIT->LAST_ACK->CLOSED

        在建立连接的时候，服务器端是在第三次握手之后才进入数据交互状态，而关闭连接则是在关闭连接的第二次握手以后（注意不是第四次）。而关闭以后还要等待客户端给出最后的ACK包才能进入初始的状态。

#TCP的11种状态
https://www.cnblogs.com/qingergege/p/6603488.html

#CLOSE_WAIT
##关于close_wait状态的理解
https://blog.csdn.net/qq_39382769/article/details/90703382

以web服务器产生大量的close_wait状态为例

1.close_wait状态介绍：
客户端主动关闭连接，服务器接收到客户端的FIN，但是还没有发送自己的FIN，此时的状态为close_wait状态，大量的close_wait状态拖累服务器性能

2.close_wait产生的原因：
某种情况下客户端关闭了连接，但是我方忙于读写，没有关闭连接

3.解决方法：

思想：检查出客户端已经关闭的连接，关闭他
之所以会出现这种问题，肯定是服务器端的连接释放的代码存在问题
1.当服务器读写失败时，可以选择关闭连接
2.定期向连接发送询问数据，检查收到的回复数据包（Heart-Beat线程发送指定格式的心跳数据包）
3.修改keep-live参数(超时时间，tcp检查间隔时间:keeplive探测包发送的间隔，tcp检查次数:如果对方不予应答，探测包发送的次数）


##线上大量CLOSE_WAIT原因排查
https://segmentfault.com/a/1190000017313251

理一下正确的分析思路：

出现问题后，立马应该检查日志，确实日志没有发现问题；

监控明确显示了socket不断增长，很明确立马应该使用 netstat 检查情况看看是哪个进程的锅；
根据 netstat 的检查，使用 tcpdump 抓包分析一下为什么连接会被动断开（TCP知识非常重要）；

如果熟悉代码应该直接去检查业务代码，如果不熟悉则可以使用 perf 把代码的调用链路打印出来；
不论是分析代码还是火焰图，到此应该能够很快定位到问题。

那么本次到底是为什么会出现  CLOSE_WAIT 呢？大部分同学应该已经明白了，我这里再简单说明一下：

由于那一行代码没有对事务进行回滚，导致服务端没有主动发起close。因此 MySQL负载均衡器 在达到 60s 的时候主动触发了close操作，但是通过tcp抓包发现，服务端并没有进行回应，这是因为代码中的事务没有处理，因此从而导致大量的端口、连接资源被占用。在贴一下挥手时的抓包数据：

 mysql 主动断开链接
11:38:45.693382 IP 172.18.0.3.3306 > 172.18.0.5.38822: Flags [F.], seq 123, ack 144, win 227, options [nop,nop,TS val 3000355 ecr 2997359], length 0 # MySQL负载均衡器发送fin包给我
11:38:45.740958 IP 172.18.0.5.38822 > 172.18.0.3.3306: Flags [.], ack 124, win 229, options [nop,nop,TS val 3000360 ecr 3000355], length 0 # 我回复ack给它

#TIME_WAIT
##谈谈 TCP 的 TIME_WAIT
https://www.cnblogs.com/zhenbianshu/p/10637964.html
由来
最近有同事在用 ab 进行服务压测，到 QPS 瓶颈后怀疑是起压机的问题，来跟我借测试机，于是我就趁机分析了一波起压机可能成为压测瓶颈的可能，除了网络 I/O、机器性能外，还考虑到了网络协议的问题。

当然本文的主角并不是压测，后来分析证明同事果然还是想多了，瓶颈是在服务端。

分析起压机瓶颈的过程中，对于 TCP TIME_WAIT 状态的一个猜想引起了我的兴趣。由于之前排查问题时，简单地接触过这个状态，但并未深入了解，于是决定抽时间分析一下，拆解一下我的猜想。

TIME_WAIT
定义
我们从上面的图中可以看出来，当 TCP 连接主动关闭时，都会经过 TIME_WAIT 状态。而且我们在机器上 curl 一个 url 创建一个 TCP 连接后，使用 ss 等工具可以在一定时长内持续观察到这个连续处于 TIME_WAIT 状态。

所以TIME_WAIT 是这么一种状态：TCP 四次握手结束后，连接双方都不再交换消息，但主动关闭的一方保持这个连接在一段时间内不可用。

那么，保持这么一个状态有什么用呢？

原因
上文中提到过，对于复杂的网络状态，TCP 的实现提出了多种应对措施，TIME_WAIT 状态的提出就是为了应对其中一种异常状况。
为了理解 TIME_WAIT 状态的必要性，我们先来假设没有这么一种状态会导致的问题。暂以 A、B 来代指 TCP 连接的两端，A 为主动关闭的一端。
四次挥手中，A 发 FIN， B 响应 ACK，B 再发 FIN，A 响应 ACK 实现连接的关闭。而如果 A 响应的 ACK 包丢失，B 会以为 A 没有收到自己的关闭请求，然后会重试向 A 再发 FIN 包。
如果没有 TIME_WAIT 状态，A 不再保存这个连接的信息，收到一个不存在的连接的包，A 会响应 RST 包，导致 B 端异常响应。

此时， TIME_WAIT 是为了保证全双工的 TCP 连接正常终止。

我们还知道，TCP 下的 IP 层协议是无法保证包传输的先后顺序的。如果双方挥手之后，一个网络四元组（src/dst ip/port）被回收，而此时网络中还有一个迟到的数据包没有被 B 接收，A 应用程序又立刻使用了同样的四元组再创建了一个新的连接后，这个迟到的数据包才到达 B，那么这个数据包就会让 B 以为是 A 刚发过来的。
此时， TIME_WAIT 的存在是为了保证网络中迷失的数据包正常过期。
由以上两个原因，TIME_WAIT 状态的存在是非常有意义的。

时长的确定
由原因来推实现，TIME_WAIT 状态的保持时长也就可以理解了。确定 TIME_WAIT 的时长主要考虑上文的第二种情况，保证关闭连接后这个连接在网络中的所有数据包都过期。
说到过期时间，不得不提另一个概念: 最大分段寿命（MSL, Maximum Segment Lifetime），它表示一个 TCP 分段可以存在于互联网系统中的最大时间，由 TCP 的实现，超出这个寿命的分片都会被丢弃。
TIME_WAIT 状态由主动关闭的 A 来保持，那么我们来考虑对于 A 来说，可能接到上一个连接的数据包的最大时长：A 刚发出的数据包，能保持 MSL 时长的寿命，它到了 B 端后，B 端由于关闭连接了，会响应 RST 包，这个 RST 包最长也会在 MSL 时长后到达 A，那么 A 端只要保持 TIME_WAIT 到达 2MS 就能保证网络中这个连接的包都会消失。
MSL 的时长被 RFC 定义为 2分钟，但在不同的 unix 实现上，这个值不并确定，我们常用的 centOS 上，它被定义为 30s，我们可以通过 /proc/sys/net/ipv4/tcp_fin_timeout 这个文件查看和修改这个值。

ab 的”奇怪”表现
猜想
由上文，我们知道由于 TIME_WAIT 的存在，每个连接被主动关闭后，这个连接就要保留 2MSL（60s） 时长，一个网络四元组也要被冻结 60s。而我们机器默认可被分配的端口号约有 30000 个（可通过 /proc/sys/net/ipv4/ip_local_port_range文件查看）。

那么如果我们使用 curl 对服务器请求时，作为客户端，都要使用本机的一个端口号，所有的端口号分配到 60s 内，每秒就要控制在 500 QPS，再多了，系统就无法再分配端口号了。

可是在使用 ab 进行压测时时，以每秒 4000 的 QPS 运行几分钟，起压机照样正常工作，使用 ss 查看连接详情时，发现一个 TIME_WAIT 状态的连接都没有。

分析
一开始我以为是 ab 使用了连接复用等技术，仔细查看了 ss 的输出发现本地端口号一直在变，到底是怎么回事呢？
于是，我在一台测试机启动了一个简单的服务，端口号 8090，然后在另一台机器上起压，并同时用 tcpdump 抓包。
结果发现，第一个 FIN 包都是由服务器发送的，即 ab 不会主动关闭连接。

登上服务器一看，果然，有大量的 TIME_WAIT 状态的连接。

但是由于服务器监听的端口会复用，这些 TIME_WAIT 状态的连接并不会对服务器造成太大影响，只是会占用一些系统资源。

小结
当然，高并发情况下，太多的 TIME_WAIT 也会给服务器造成很大的压力，毕竟维护这么多 socket 也是要消耗资源的，关于如何解决 TIME_WAIT 过多的问题，可以看 tcp短连接TIME_WAIT问题解决方法大全（1）——高屋建瓴。

##tcp短连接TIME_WAIT问题解决方法大全（1）——高屋建瓴
https://blog.csdn.net/yunhua_lee/article/details/8146830
 本文主要关注如何解决tcp短连接的TIME_WAIT问题。


短连接最大的优点是方便，特别是脚本语言，由于执行完毕后脚本语言的进程就结束了，基本上都是用短连接。
但短连接最大的缺点是将占用大量的系统资源，例如：本地端口、socket句柄。
导致这个问题的原因其实很简单：tcp协议层并没有长短连接的概念，因此不管长连接还是短连接，连接建立->数据传输->连接关闭的流程和处理都是一样的。


正常的TCP客户端连接在关闭后，会进入一个TIME_WAIT的状态，持续的时间一般在1~4分钟，对于连接数不高的场景，1~4分钟其实并不长，对系统也不会有什么影响，
但如果短时间内（例如1s内）进行大量的短连接，则可能出现这样一种情况：客户端所在的操作系统的socket端口和句柄被用尽，系统无法再发起新的连接！


举例来说：假设每秒建立了1000个短连接（Web场景下是很常见的，例如每个请求都去访问memcached），假设TIME_WAIT的时间是1分钟，则1分钟内需要建立6W个短连接，
由于TIME_WAIT时间是1分钟，这些短连接1分钟内都处于TIME_WAIT状态，都不会释放，而Linux默认的本地端口范围配置是：net.ipv4.ip_local_port_range = 32768    61000
不到3W，因此这种情况下新的请求由于没有本地端口就不能建立了。


可以通过如下方式来解决这个问题：
1）可以改为长连接，但代价较大，长连接太多会导致服务器性能问题，而且PHP等脚本语言，需要通过proxy之类的软件才能实现长连接；
2）修改ipv4.ip_local_port_range，增大可用端口范围，但只能缓解问题，不能根本解决问题；
3）客户端程序中设置socket的SO_LINGER选项；
4）客户端机器打开tcp_tw_recycle和tcp_timestamps选项；
5）客户端机器打开tcp_tw_reuse和tcp_timestamps选项；
6）客户端机器设置tcp_max_tw_buckets为一个很小的值；


在解决php连接Memcached的短连接问题过程中，我们主要验证了3）4）5）6）几种方法，采取的是基本功能验证和代码验证，并没有进行性能压力测试验证，
因此实际应用的时候需要注意观察业务运行情况，发现丢包、断连、无法连接等现象时，需要关注是否是因为这些选项导致的。 


##tcp短连接TIME_WAIT问题解决方法大全（2）——SO_LINGER
https://blog.csdn.net/yah99_wolf/article/details/8146837
 SO_LINGER是一个socket选项，通过setsockopt API进行设置，使用起来比较简单，但其实现机制比较复杂，且字面意思上比较难理解。
解释最清楚的当属《Unix网络编程卷1》中的说明（7.5章节），这里简单摘录：
SO_LINGER的值用如下数据结构表示：
struct linger {
     int l_onoff; /* 0 = off, nozero = on */
     int l_linger; /* linger time */
};

其取值和处理如下：
1、设置 l_onoff为0，则该选项关闭，l_linger的值被忽略，等于内核缺省情况，close调用会立即返回给调用者，如果可能将会传输任何未发送的数据；
2、设置 l_onoff为非0，l_linger为0，则套接口关闭时TCP夭折连接，TCP将丢弃保留在套接口发送缓冲区中的任何数据并发送一个RST给对方，
   而不是通常的四分组终止序列，这避免了TIME_WAIT状态；
3、设置 l_onoff 为非0，l_linger为非0，当套接口关闭时内核将拖延一段时间（由l_linger决定）。
   如果套接口缓冲区中仍残留数据，进程将处于睡眠状态，直 到（a）所有数据发送完且被对方确认，之后进行正常的终止序列（描述字访问计数为0）
   或（b）延迟时间到。此种情况下，应用程序检查close的返回值是非常重要的，如果在数据发送完并被确认前时间到，close将返回EWOULDBLOCK错误且套接口发送缓冲区中的任何数据都丢失。
   close的成功返回仅告诉我们发送的数据（和FIN）已由对方TCP确认，它并不能告诉我们对方应用进程是否已读了数据。如果套接口设为非阻塞的，它将不等待close完成。
   
第一种情况其实和不设置没有区别，第二种情况可以用于避免TIME_WAIT状态，但在Linux上测试的时候，并未发现发送了RST选项，而是正常进行了四步关闭流程，
初步推断是“只有在丢弃数据的时候才发送RST”，如果没有丢弃数据，则走正常的关闭流程。
查看Linux源码，确实有这么一段注释和源码：
=====linux-2.6.37 net/ipv4/tcp.c 1915=====
/* As outlined in RFC 2525, section 2.17, we send a RST here because
* data was lost. To witness the awful effects of the old behavior of
* always doing a FIN, run an older 2.1.x kernel or 2.0.x, start a bulk
* GET in an FTP client, suspend the process, wait for the client to
* advertise a zero window, then kill -9 the FTP client, wheee...
* Note: timeout is always zero in such a case.
*/
if (data_was_unread) {
/* Unread data was tossed, zap the connection. */
NET_INC_STATS_USER(sock_net(sk), LINUX_MIB_TCPABORTONCLOSE);
tcp_set_state(sk, TCP_CLOSE);
tcp_send_active_reset(sk, sk->sk_allocation);
} 
另外，从原理上来说，这个选项有一定的危险性，可能导致丢数据，使用的时候要小心一些，但我们在实测libmemcached的过程中，没有发现此类现象，
应该是和libmemcached的通讯协议设置有关，也可能是我们的压力不够大，不会出现这种情况。


第三种情况其实就是第一种和第二种的折中处理，且当socket为非阻塞的场景下是没有作用的。
对于应对短连接导致的大量TIME_WAIT连接问题，个人认为第二种处理是最优的选择，libmemcached就是采用这种方式，
从实测情况来看，打开这个选项后，TIME_WAIT连接数为0，且不受网络组网（例如是否虚拟机等）的影响。
 
##tcp短连接TIME_WAIT问题解决方法大全（3）——tcp_tw_recycle
原文链接：https://blog.csdn.net/yah99_wolf/java/article/details/8146845
【tcp_tw_recycle和tcp_timestamps】
参考官方文档（http://www.kernel.org/doc/Documentation/networking/ip-sysctl.txt），tcp_tw_recycle解释如下：
tcp_tw_recycle选项作用为：Enable fast recycling TIME-WAIT sockets. Default value is 0.
tcp_timestamps选项作用为：Enable timestamps as defined in RFC1323. Default value is 1.

这两个选项是linux内核提供的控制选项，和具体的应用程序没有关系，而且网上也能够查询到大量的相关资料，但信息都不够完整，最主要的几个问题如下；
1）快速回收到底有多快？
2）有的资料说只要打开tcp_tw_recycle即可，有的又说要tcp_timestamps同时打开，具体是哪个正确？
3）为什么从虚拟机NAT出去发起客户端连接时选项无效，非虚拟机连接就有效？

为了回答上面的疑问，只能看代码，看出一些相关的代码供大家参考：
=====linux-2.6.37 net/ipv4/tcp_minisocks.c 269======
void tcp_time_wait(struct sock *sk, int state, int timeo)
{
struct inet_timewait_sock *tw = NULL;
const struct inet_connection_sock *icsk = inet_csk(sk);
const struct tcp_sock *tp = tcp_sk(sk);
int recycle_ok = 0;


    //判断是否快速回收，这里可以看出tcp_tw_recycle和tcp_timestamps两个选项都打开的时候才进行快速回收，
    //且还有进一步的判断条件，后面会分析，这个进一步的判断条件和第三个问题有关
if (tcp_death_row.sysctl_tw_recycle && tp->rx_opt.ts_recent_stamp)
recycle_ok = icsk->icsk_af_ops->remember_stamp(sk);


if (tcp_death_row.tw_count < tcp_death_row.sysctl_max_tw_buckets)
tw = inet_twsk_alloc(sk, state);


if (tw != NULL) {
struct tcp_timewait_sock *tcptw = tcp_twsk((struct sock *)tw);
        //计算快速回收的时间，等于 RTO * 3.5，回答第一个问题的关键是RTO（Retransmission Timeout）大概是多少
const int rto = (icsk->icsk_rto << 2) - (icsk->icsk_rto >> 1);
        
        //。。。。。。此处省略很多代码。。。。。。
        
    if (recycle_ok) {
            //设置快速回收的时间
tw->tw_timeout = rto;
} else {
tw->tw_timeout = TCP_TIMEWAIT_LEN;
if (state == TCP_TIME_WAIT)
timeo = TCP_TIMEWAIT_LEN;
}
        
        //。。。。。。此处省略很多代码。。。。。。
}


RFC中有关于RTO计算的详细规定，一共有三个：RFC-793、RFC-2988、RFC-6298，Linux的实现是参考RFC-2988。
对于这些算法的规定和Linuxde 实现，有兴趣的同学可以自己深入研究，实际应用中我们只要记住Linux如下两个边界值：
=====linux-2.6.37 net/ipv4/tcp.c 126================
==#define TCP_RTO_MAX ((unsigned)(120*HZ))
==#define TCP_RTO_MIN ((unsigned)(HZ/5))
==========================================
这里的HZ是1s，因此可以得出RTO最大是120s，最小是200ms，对于局域网的机器来说，正常情况下RTO基本上就是200ms，因此3.5 RTO就是700ms
也就是说，快速回收是TIME_WAIT的状态持续700ms，而不是正常的2MSL（Linux是1分钟，请参考：include/net/tcp.h 109行TCP_TIMEWAIT_LEN定义）。
实测结果也验证了这个推论，不停的查看TIME_WAIT状态的连接，偶尔能看到1个。

最后一个问题是为什么从虚拟机发起的连接即使设置了tcp_tw_recycle和tcp_timestamps，也不会快速回收，继续看代码：
tcp_time_wait函数中的代码行：recycle_ok = icsk->icsk_af_ops->remember_stamp(sk);对应的实现如下：
=====linux-2.6.37 net/ipv4/tcp_ipv4.c 1772=====
int tcp_v4_remember_stamp(struct sock *sk)
{
    //。。。。。。此处省略很多代码。。。。。。


    //当获取对端信息时，进行快速回收，否则不进行快速回收
if (peer) {
if ((s32)(peer->tcp_ts - tp->rx_opt.ts_recent) <= 0 ||
   ((u32)get_seconds() - peer->tcp_ts_stamp > TCP_PAWS_MSL &&
    peer->tcp_ts_stamp <= (u32)tp->rx_opt.ts_recent_stamp)) {
peer->tcp_ts_stamp = (u32)tp->rx_opt.ts_recent_stamp;
peer->tcp_ts = tp->rx_opt.ts_recent;
}
if (release_it)
inet_putpeer(peer);
return 1;
}


return 0;
}
上面这段代码应该就是测试的时候虚拟机环境不会释放的原因，当使用虚拟机NAT出去的时候，服务器无法获取隐藏在NAT后的机器信息。
生产环境也出现了设置了选项，但TIME_WAIT连接数达到4W多的现象，可能和虚拟机有关，也可能和组网有关。


总结一下：
1）快速回收到底有多快？
局域网环境下，700ms就回收；
2）有的资料说只要打开tcp_tw_recycle即可，有的又说要tcp_timestamps同时打开，具体是哪个正确？
需要同时打开，但默认情况下tcp_timestamps就是打开的，所以会有人说只要打开tcp_tw_recycle即可；
3）为什么从虚拟机发起客户端连接时选项无效，非虚拟机连接就有效？
和网络组网有关系，无法获取对端信息时就不进行快速回收；

综合上面的分析和总结，可以看出这种方法不是很保险，在实际应用中可能受到虚拟机、网络组网、防火墙之类的影响从而导致不能进行快速回收。

附：
1）tcp_timestamps的说明详见RF1323，和TCP的拥塞控制（Congestion control）有关。

2）打开此选项，可能导致无法连接，请参考：http://www.pagefault.info/?p=416 

##tcp短连接TIME_WAIT问题解决方法大全（4）——tcp_tw_reuse
https://blog.csdn.net/yah99_wolf/java/article/details/8146856
tcp_tw_reuse选项的含义如下（http://www.kernel.org/doc/Documentation/networking/ip-sysctl.txt）：
tcp_tw_reuse - BOOLEAN
Allow to reuse TIME-WAIT sockets for new connections when it is
safe from protocol viewpoint. Default value is 0.
    
这里的关键在于“协议什么情况下认为是安全的”，由于环境限制，没有办法进行验证，通过看源码简单分析了一下。
=====linux-2.6.37 net/ipv4/tcp_ipv4.c 114=====
int tcp_twsk_unique(struct sock *sk, struct sock *sktw, void *twp)
{
const struct tcp_timewait_sock *tcptw = tcp_twsk(sktw);
struct tcp_sock *tp = tcp_sk(sk);


/* With PAWS, it is safe from the viewpoint
  of data integrity. Even without PAWS it is safe provided sequence
  spaces do not overlap i.e. at data rates <= 80Mbit/sec.


  Actually, the idea is close to VJ's one, only timestamp cache is
  held not per host, but per port pair and TW bucket is used as state
  holder.


  If TW bucket has been already destroyed we fall back to VJ's scheme
  and use initial timestamp retrieved from peer table.
*/
    //从代码来看，tcp_tw_reuse选项和tcp_timestamps选项也必须同时打开；否则tcp_tw_reuse就不起作用
    //另外，所谓的“协议安全”，从代码来看应该是收到最后一个包后超过1s
if (tcptw->tw_ts_recent_stamp &&
   (twp == NULL || (sysctl_tcp_tw_reuse &&
    get_seconds() - tcptw->tw_ts_recent_stamp > 1))) {
tp->write_seq = tcptw->tw_snd_nxt + 65535 + 2;
if (tp->write_seq == 0)
tp->write_seq = 1;
tp->rx_opt.ts_recent  = tcptw->tw_ts_recent;
tp->rx_opt.ts_recent_stamp = tcptw->tw_ts_recent_stamp;
sock_hold(sktw);
return 1;
}


return 0;
}



总结一下：
1）tcp_tw_reuse选项和tcp_timestamps选项也必须同时打开；
2）重用TIME_WAIT的条件是收到最后一个包后超过1s。


官方手册有一段警告：
It should not be changed without advice/request of technical
experts.
对于大部分局域网或者公司内网应用来说，满足条件2）都是没有问题的，因此官方手册里面的警告其实也没那么可怕：）

##tcp短连接TIME_WAIT问题解决方法大全（5）——tcp_max_tw_buckets
https://blog.csdn.net/yah99_wolf/java/article/details/8146862

参考官方文档（http://www.kernel.org/doc/Documentation/networking/ip-sysctl.txt），解释如下：
tcp_max_tw_buckets - INTEGER
Maximal number of timewait sockets held by system simultaneously.
If this number is exceeded time-wait socket is immediately destroyed
and warning is printed. 
官方文档没有说明默认值，通过几个系统的简单验证，初步确定默认值是180000。

通过源码查看发现，这个选项比较简单，其实现代码如下：
=====linux-2.6.37 net/ipv4/tcp_minisocks.c 269======
void tcp_time_wait(struct sock *sk, int state, int timeo)
{
struct inet_timewait_sock *tw = NULL;
const struct inet_connection_sock *icsk = inet_csk(sk);
const struct tcp_sock *tp = tcp_sk(sk);
int recycle_ok = 0;


if (tcp_death_row.sysctl_tw_recycle && tp->rx_opt.ts_recent_stamp)
recycle_ok = icsk->icsk_af_ops->remember_stamp(sk);


if (tcp_death_row.tw_count < tcp_death_row.sysctl_max_tw_buckets)
tw = inet_twsk_alloc(sk, state);


if (tw != NULL) {
        //分配成功，进行TIME_WAIT状态处理，此处略去很多代码
    else {
        //分配失败，不进行处理，只记录日志: TCP: time wait bucket table overflow
/* Sorry, if we're out of memory, just CLOSE this
* socket up.  We've got bigger problems than
* non-graceful socket closings.
*/
NET_INC_STATS_BH(sock_net(sk), LINUX_MIB_TCPTIMEWAITOVERFLOW);
}


tcp_update_metrics(sk);
tcp_done(sk);
}
实测结果验证，配置为100，TIME_WAIT连接数就稳定在100，且不受组网和其它配置的影响。

官方手册中有一段警告：
    This limit exists only to prevent
simple DoS attacks, you _must_ not lower the limit artificially,
but rather increase it (probably, after increasing installed memory),
if network conditions require more than default value.
基本意思是这个用于防止Dos攻击，我们不应该人工减少，如果网络条件需要的话，反而应该增加。
但其实对于我们的局域网或者公司内网应用来说，这个风险并不大。



#三次握手和四次挥手

##TCP的三次握手与四次挥手理解及面试题
https://blog.csdn.net/qq_38950316/article/details/81087809

 本文经过借鉴书籍资料、他人博客总结出的知识点，欢迎提问



    序列号seq：占4个字节，用来标记数据段的顺序，TCP把连接中发送的所有数据字节都编上一个序号，第一个字节的编号由本地随机产生；给字节编上序号后，就给每一个报文段指派一个序号；序列号seq就是这个报文段中的第一个字节的数据编号。

    确认号ack：占4个字节，期待收到对方下一个报文段的第一个数据字节的序号；序列号表示报文段携带数据的第一个字节的编号；而确认号指的是期望接收到下一个字节的编号；因此当前报文段最后一个字节的编号+1即为确认号。

    确认ACK：占1位，仅当ACK=1时，确认号字段才有效。ACK=0时，确认号无效

    同步SYN：连接建立时用于同步序号。当SYN=1，ACK=0时表示：这是一个连接请求报文段。若同意连接，则在响应报文段中使得SYN=1，ACK=1。因此，SYN=1表示这是一个连接请求，或连接接受报文。SYN这个标志位只有在TCP建产连接时才会被置1，握手完成后SYN标志位被置0。

    终止FIN：用来释放一个连接。FIN=1表示：此报文段的发送方的数据已经发送完毕，并要求释放运输连接

    PS：ACK、SYN和FIN这些大写的单词表示标志位，其值要么是1，要么是0；ack、seq小写的单词表示序号。

 
字段	含义
URG	紧急指针是否有效。为1，表示某一位需要被优先处理
ACK	确认号是否有效，一般置为1。
PSH	提示接收端应用程序立即从TCP缓冲区把数据读走。
RST	对方要求重新建立连接，复位。
SYN	请求建立连接，并在其序列号的字段进行序列号的初始值设定。建立连接，设置为1
FIN    	希望断开连接。
三次握手过程理解


 

第一次握手：建立连接时，客户端发送syn包（syn=x）到服务器，并进入SYN_SENT状态，等待服务器确认；SYN：同步序列编号（Synchronize Sequence Numbers）。

第二次握手：服务器收到syn包，必须确认客户的SYN（ack=x+1），同时自己也发送一个SYN包（syn=y），即SYN+ACK包，此时服务器进入SYN_RECV状态；

第三次握手：客户端收到服务器的SYN+ACK包，向服务器发送确认包ACK(ack=y+1），此包发送完毕，客户端和服务器进入ESTABLISHED（TCP连接成功）状态，完成三次握手。

四次挥手过程理解 


1）客户端进程发出连接释放报文，并且停止发送数据。释放数据报文首部，FIN=1，其序列号为seq=u（等于前面已经传送过来的数据的最后一个字节的序号加1），此时，客户端进入FIN-WAIT-1（终止等待1）状态。 TCP规定，FIN报文段即使不携带数据，也要消耗一个序号。
2）服务器收到连接释放报文，发出确认报文，ACK=1，ack=u+1，并且带上自己的序列号seq=v，此时，服务端就进入了CLOSE-WAIT（关闭等待）状态。TCP服务器通知高层的应用进程，客户端向服务器的方向就释放了，这时候处于半关闭状态，即客户端已经没有数据要发送了，但是服务器若发送数据，客户端依然要接受。这个状态还要持续一段时间，也就是整个CLOSE-WAIT状态持续的时间。
3）客户端收到服务器的确认请求后，此时，客户端就进入FIN-WAIT-2（终止等待2）状态，等待服务器发送连接释放报文（在这之前还需要接受服务器发送的最后的数据）。
4）服务器将最后的数据发送完毕后，就向客户端发送连接释放报文，FIN=1，ack=u+1，由于在半关闭状态，服务器很可能又发送了一些数据，假定此时的序列号为seq=w，此时，服务器就进入了LAST-ACK（最后确认）状态，等待客户端的确认。
5）客户端收到服务器的连接释放报文后，必须发出确认，ACK=1，ack=w+1，而自己的序列号是seq=u+1，此时，客户端就进入了TIME-WAIT（时间等待）状态。注意此时TCP连接还没有释放，必须经过2∗∗MSL（最长报文段寿命）的时间后，当客户端撤销相应的TCB后，才进入CLOSED状态。
6）服务器只要收到了客户端发出的确认，立即进入CLOSED状态。同样，撤销TCB后，就结束了这次的TCP连接。可以看到，服务器结束TCP连接的时间要比客户端早一些。

 常见面试题
【问题1】为什么连接的时候是三次握手，关闭的时候却是四次握手？

答：因为当Server端收到Client端的SYN连接请求报文后，可以直接发送SYN+ACK报文。其中ACK报文是用来应答的，SYN报文是用来同步的。但是关闭连接时，当Server端收到FIN报文时，很可能并不会立即关闭SOCKET，所以只能先回复一个ACK报文，告诉Client端，"你发的FIN报文我收到了"。只有等到我Server端所有的报文都发送完了，我才能发送FIN报文，因此不能一起发送。故需要四步握手。

【问题2】为什么TIME_WAIT状态需要经过2MSL(最大报文段生存时间)才能返回到CLOSE状态？

答：虽然按道理，四个报文都发送完毕，我们可以直接进入CLOSE状态了，但是我们必须假象网络是不可靠的，有可以最后一个ACK丢失。所以TIME_WAIT状态就是用来重发可能丢失的ACK报文。在Client发送出最后的ACK回复，但该ACK可能丢失。Server如果没有收到ACK，将不断重复发送FIN片段。所以Client不能立即关闭，它必须确认Server接收到了该ACK。Client会在发送出ACK之后进入到TIME_WAIT状态。Client会设置一个计时器，等待2MSL的时间。如果在该时间内再次收到FIN，那么Client会重发ACK并再次等待2MSL。所谓的2MSL是两倍的MSL(Maximum Segment Lifetime)。MSL指一个片段在网络中最大的存活时间，2MSL就是一个发送和一个回复所需的最大时间。如果直到2MSL，Client都没有再次收到FIN，那么Client推断ACK已经被成功接收，则结束TCP连接。

【问题3】为什么不能用两次握手进行连接？

答：3次握手完成两个重要的功能，既要双方做好发送数据的准备工作(双方都知道彼此已准备好)，也要允许双方就初始序列号进行协商，这个序列号在握手过程中被发送和确认。

       现在把三次握手改成仅需要两次握手，死锁是可能发生的。作为例子，考虑计算机S和C之间的通信，假定C给S发送一个连接请求分组，S收到了这个分组，并发 送了确认应答分组。按照两次握手的协定，S认为连接已经成功地建立了，可以开始发送数据分组。可是，C在S的应答分组在传输中被丢失的情况下，将不知道S 是否已准备好，不知道S建立什么样的序列号，C甚至怀疑S是否收到自己的连接请求分组。在这种情况下，C认为连接还未建立成功，将忽略S发来的任何数据分 组，只等待连接确认应答分组。而S在发出的分组超时后，重复发送同样的分组。这样就形成了死锁。

【问题4】如果已经建立了连接，但是客户端突然出现故障了怎么办？

TCP还设有一个保活计时器，显然，客户端如果出现故障，服务器不能一直等下去，白白浪费资源。服务器每收到一次客户端的请求后都会重新复位这个计时器，时间通常是设置为2小时，若两小时还没有收到客户端的任何数据，服务器就会发送一个探测报文段，以后每隔75秒钟发送一次。若一连发送10个探测报文仍然没反应，服务器就认为客户端出了故障，接着就关闭连接。


##TCP：WireShark分析，序列号Seq和确认号Ack
https://www.cnblogs.com/lidabo/p/9466844.htm##理解TCP序列号（Sequence Number）和确认号（Acknowledgment Number）
https://www.cnblogs.com/QingFlye/p/4442529.html

##TCP 握手和挥手图解（有限状态机）
https://blog.csdn.net/xy010902100449/article/details/48274635

（1）CLOSED 状态时初始状态。

（2）LISTEN:被动打开，服务器端的 状态变为LISTEN(监听)。被动打开的概念：连接的一端的应用程序通知操作系统，希望建立一个传入的连接。这时候操作系统为连接的这一端建立一个连 接。与之对应的是主动连接：应用程序通过主动打开请求来告诉操作系统建立一个连接。

（3）SYNRECVD:服务器端收到SYN后，状态为SYN；发送SYN ACK;

（4）SYN_SENTY:应用程序发送SYN后，状态为SYN_SENT；

（5）ESTABLISHED:SYNRECVD收到ACK后，状态为ESTABLISHED； SYN_SENT在收到SYN ACK，发送ACK，状态为ESTABLISHED；

（6）CLOSE_WAIT:服务器端在收到FIN后，发送ACK，状态为CLOSE_WAIT；如果此时服务器端还有数据需要发送，那么就发送，直到数据发送完毕；此时，服务器端发送FIN，状态变为LAST_ACK;

（7）FIN_WAIT_1：应用程序端发送FIN，准备断开TCP连接；状态从ESTABLISHED——>FIN_WAIT_1；

（8）FIN_WAIT_2：应用程序端只收到服务器端得ACK信号，并没有收到FIN信号；说明服务器端还有数据传输，那么此时为半连接；

（9）TIME_WAIT:有两种方式进入 该状态：1、FIN_WAIT_1进入：此时应用程序端口收到FIN+ACK（而不是像FIN_WAIT_2那样只收到ACK，说明数据已经发送完毕）并 向服务器端口发送ACK；2、FIN_WAIT_2进入：此时应用程序端口收到了FIN，然后向服务器端发送ACK；TIME_WAIT是为了实现TCP 全双工连接的可靠性关闭，用来重发可能丢失的ACK报文；需要持续2个MSL(最大报文生存时间)：假设应用程序端口在进入TIME_WAIT后，2个 MSL时间内并没有收到FIN,说明应用程序最后发出的ACK已经收到了；否则，会在2个MSL内在此收到ACK报文；

4.1.客户端应用程序的状态迁移图
客户端的状态可以用如下的流程来表示：
CLOSED->SYN_SENT->ESTABLISHED->FIN_WAIT_1->FIN_WAIT_2->TIME_WAIT->CLOSED
以上流程是在程序正常的情况下应该有的流程，从书中的图中可以看到，在建立连接时，当客户端收到SYN报文的ACK以后，客户端就打开了数据交互地连接。而结束连接则通常是客户端主动结束的，客户端结束应用程序以后，需要经历FIN_WAIT_1，FIN_WAIT_2等状态，这些状态的迁移就是前面提到的结束连接的四次握手。

4.2.服务器的状态迁移图
服务器的状态可以用如下的流程来表示：
CLOSED->LISTEN->SYN收到->ESTABLISHED->CLOSE_WAIT->LAST_ACK->CLOSED
在建立连接的时候，服务器端是在第三次握手之后才进入数据交互状态，而关闭连接则是在关闭连接的第二次握手以后（注意不是第四次）。而关闭以后还要等待客户端给出最后的ACK包才能进入初始的状态。

4.3.其他状态迁移
书中的图还有一些其他的状态迁移，这些状态迁移针对服务器和客户端两方面的总结如下
LISTEN->SYN_SENT，对于这个解释就很简单了，服务器有时候也要打开连接的嘛。
SYN_SENT->SYN收到，服务器和客户端在SYN_SENT状态下如果收到SYN数据报，则都需要发送SYN的ACK数据报并把自己的状态调整到SYN收到状态，准备进入ESTABLISHED
SYN_SENT->CLOSED，在发送超时的情况下，会返回到CLOSED状态。
SYN_收到->LISTEN，如果受到RST包，会返回到LISTEN状态。
SYN_收到->FIN_WAIT_1，这个迁移是说，可以不用到ESTABLISHED状态，而可以直接跳转到FIN_WAIT_1状态并等待关闭。

4.4.2MSL等待状态
书中给的图里面，有一个TIME_WAIT等待状态，这个状态又叫做2MSL状态，说的是在TIME_WAIT2发送了最后一个ACK数据报以后，要进入TIME_WAIT状态，这个状态是防止最后一次握手的数据报没有传送到对方那里而准备的（注意这不是四次握手，这是第四次握手的保险状态）。这个状态在很大程度上保证了双方都可以正常结束，但是，问题也来了。

由于插口的2MSL状态（插口是IP和端口对的意思，socket），使得应用程序在2MSL时间内是无法再次使用同一个插口的，对于客户程序还好一些，但是对于服务程序，例如httpd，它总是要使用同一个端口来进行服务，而在2MSL时间内，启动httpd就会出现错误（插口被使用）。为了避免这个错误，服务器给出了一个平静时间的概念，这是说在2MSL时间内，虽然可以重新启动服务器，但是这个服务器还是要平静的等待2MSL时间的过去才能进行下一次连接。

4.5.FIN_WAIT_2状态
这就是著名的半关闭的状态了，这是在关闭连接时，客户端和服务器两次握手之后的状态。在这个状态下，应用程序还有接受数据的能力，但是已经无法发送数据，但是也有一种可能是，客户端一直处于FIN_WAIT_2状态，而服务器则一直处于WAIT_CLOSE状态，而直到应用层来决定关闭这个状态。
5.RST，同时打开和同时关闭
RST是另一种关闭连接的方式，应用程序应该可以判断RST包的真实性，即是否为异常中止。而同时打开和同时关闭则是两种特殊的TCP状态，发生的概率很小。

##硬核！近 40 张图解被问千百遍的 TCP 三次握手和四次挥手面试题 
https://www.sohu.com/a/390795443_106784

序列号：在建立连接时由计算机生成的随机数作为其初始值，通过 SYN 包传给接收端主机，每发送一次数据，就「累加」一次该「数据字节数」的大小。 用来解决网络包乱序问题。

确认应答号：指下一次「期望」收到的数据的序列号，发送端收到这个确认应答以后可以认为在这个序号以前的数据都已经被正常接收。 用来解决不丢包的问题。

控制位：

ACK：该位为 1 时，「确认应答」的字段变为有效，TCP 规定除了最初建立连接时的 SYN 包之外该位必须设置为 1 。
RST：该位为 1 时，表示 TCP 连接中出现异常必须强制断开连接。
SYC：该位为 1 时，表示希望建立连，并在其「序列号」的字段进行序列号初始值的设定。
FIN：该位为 1 时，表示今后不会再有数据发送，希望断开连接。当通信结束希望断开连接时，通信双方的主机之间就可以相互交换 FIN 位置为 1 的 TCP 段。
为什么需要 TCP 协议？TCP 工作在哪一层？

IP 层是「不可靠」的，它不保证网络包的交付、不保证网络包的按序交付、也不保证网络包中的数据的完整性。

TCP 四元组可以唯一的确定一个连接，四元组包括如下：

源地址
源端口
目的地址
目的端口


源地址和目的地址的字段（32位）是在 IP 头部中，作用是通过 IP 协议发送报文给对方主机。

源端口和目的端口的字段（16位）是在 TCP 头部中，作用是告诉 TCP 协议应该把报文发给哪个进程。

TCP 和 UDP 区别：

1. 连接

TCP 是面向连接的传输层协议，传输数据前先要建立连接。
UDP 是不需要连接，即刻传输数据。
2. 服务对象

TCP 是一对一的两点服务，即一条连接只有两个端点。
UDP 支持一对一、一对多、多对多的交互通信
3. 可靠性

TCP 是可靠交付数据的，数据可以无差错、不丢失、不重复、按需到达。
UDP 是尽最大努力交付，不保证可靠交付数据。
4. 拥塞控制、流量控制

TCP 有拥塞控制和流量控制机制，保证数据传输的安全性。
UDP 则没有，即使网络非常拥堵了，也不会影响 UDP 的发送速率。
5. 首部开销

TCP 首部长度较长，会有一定的开销，首部在没有使用「选项」字段时是 20 个字节，如果使用了「选项」字段则会变长的。
UDP 首部只有 8 个字节，并且是固定不变的，开销较小。
TCP 和 UDP 应用场景：

由于 TCP 是面向连接，能保证数据的可靠性交付，因此经常用于：

FTP 文件传输
HTTP / HTTPS
由于 UDP 面向无连接，它可以随时发送数据，再加上UDP本身的处理既简单又高效，因此经常用于：

包总量较少的通信，如 DNS 、SNMP 等
视频、音频等多媒体通信
广播通信
为什么 UDP 头部没有「首部长度」字段，而 TCP 头部有「首部长度」字段呢？

原因是 TCP 有可变长的「选项」字段，而 UDP 头部长度则是不会变化的，无需多一个字段去记录 UDP 的首部长度。

为什么 UDP 头部有「包长度」字段，而 TCP 头部则没有「包长度」字段呢？

先说说 TCP 是如何计算负载数据长度：



其中 IP 总长度 和 IP 首部长度，在 IP 首部格式是已知的。TCP 首部长度，则是在 TCP 首部格式已知的，所以就可以求得 TCP 数据的长度。

大家这时就奇怪了问：“ UDP 也是基于 IP 层的呀，那 UDP 的数据长度也可以通过这个公式计算呀？为何还要有「包长度」呢？”

这么一问，确实感觉 UDP 「包长度」是冗余的。

如果去掉 UDP 「包长度」字段，那 UDP 首部长度就不是 4 字节的整数倍了，所以小林觉得这可能是为了补全 UDP 首部长度是 4 字节的整数倍，才补充了「包长度」字段。

为什么三次握手才可以初始化Socket、序列号和窗口大小并建立 TCP 连接。

接下来以三个方面分析三次握手的原因：

三次握手才可以阻止历史重复连接的初始化（主要原因）
三次握手才可以同步双方的初始序列号
三次握手才可以避免资源浪费

小结

TCP 建立连接时，通过三次握手 能防止历史连接的建立，能减少双方不必要的资源开销，能帮助双方同步初始化序列号。序列号能够保证数据包不重复、不丢弃和按序传输。

不使用「两次握手」和「四次握手」的原因：

「两次握手」：无法防止历史连接的建立，会造成双方资源的浪费，也无法可靠的同步双方序列号；
「四次握手」：三次握手就已经理论上最少可靠连接建立，所以不需要使用更多的通信次数。



为了达到最佳的传输效能 TCP 协议在建立连接的时候通常要协商双方的 MSS 值，当 TCP 层发现数据超过 MSS 时，则就先会进行分片，当然由它形成的 IP 包的长度也就不会大于 MTU ，自然也就不用 IP 分片了。


SYN 攻击

我们都知道 TCP 连接建立是需要三次握手，假设攻击者短时间伪造不同 IP 地址的 SYN 报文，服务端每接收到一个 SYN 报文，就进入SYN_RCVD 状态，但服务端发送出去的 ACK + SYN 报文，无法得到未知 IP 主机的 ACK 应答，久而久之就会 占满服务端的 SYN 接收队列（未连接队列），使得服务器不能为正常用户服务。

避免 SYN 攻击方式一

其中一种解决方式是通过修改 Linux 内核参数，控制队列大小和当队列满时应做什么处理。

当网卡接收数据包的速度大于内核处理的速度时，会有一个队列保存这些数据包。控制该队列的最大值如下参数：
net.core.netdev_max_backlog

SYN_RCVD 状态连接的最大个数：

net.ipv4.tcp_max_syn_backlog

超出处理能时，对新的 SYN 直接回 RST，丢弃连接：

net.ipv4.tcp_abort_on_overflow

避免 SYN 攻击方式二

我们先来看下Linux 内核的 SYN （未完成连接建立）队列与 Accpet （已完成连接建立）队列是如何工作的？

正常流程：

当服务端接收到客户端的 SYN 报文时，会将其加入到内核的「 SYN 队列」；
接着发送 SYN + ACK 给客户端，等待客户端回应 ACK 报文；
服务端接收到 ACK 报文后，从「 SYN 队列」移除放入到「 Accept 队列」；
应用通过调用 accpet socket 接口，从「 Accept 队列」取出的连接。

受到 SYN 攻击：

如果不断受到 SYN 攻击，就会导致「 SYN 队列」被占满。
tcp_syncookies 的方式可以应对 SYN 攻击的方法：

net.ipv4.tcp_syncookies = 1



tcp_syncookies 应对 SYN 攻击

当 「 SYN 队列」满之后，后续服务器收到 SYN 包，不进入「 SYN 队列」；
计算出一个 cookie 值，再以 SYN + ACK 中的「序列号」返回客户端，
服务端接收到客户端的应答报文时，服务器会检查这个 ACK 包的合法性。如果合法，直接放入到「 Accept 队列」。
最后应用通过调用 accpet socket 接口，从「 Accept 队列」取出的连接。

TCP 连接断开

TCP 四次挥手过程和状态变迁

天下没有不散的宴席，对于 TCP 连接也是这样， TCP 断开连接是通过四次挥手方式。

双方都可以主动断开连接，断开连接后主机中的「资源」将被释放。

再来回顾下四次挥手双方发 FIN 包的过程，就能理解为什么需要四次了。

关闭连接时，客户端向服务端发送 FIN 时，仅仅表示客户端不再发送数据了但是还能接收数据。
服务器收到客户端的 FIN 报文时，先回一个 ACK 应答报文，而服务端可能还有数据需要处理和发送，等服务端不再发送数据时，才发送 FIN 报文给客户端来表示同意现在关闭连接。
从上面过程可知，服务端通常需要等待完成数据的发送和处理，所以服务端的 ACK 和 FIN 一般都会分开发送，从而比三次握手导致多了一次。



需要 TIME-WAIT 状态，主要是两个原因：

防止具有相同「四元组」的「旧」数据包被收到；
保证「被动关闭连接」的一方能被正确的关闭，即保证最后的 ACK 能让被动关闭方接收，从而帮助其正常关闭；

原因一：防止旧连接的数据包


原因二：保证连接正确关闭

TIME_WAIT 过多有什么危害？

如果服务器有处于 TIME-WAIT 状态的 TCP，则说明是由服务器方主动发起的断开请求。

过多的 TIME-WAIT 状态主要的危害有两种：

第一是内存资源占用；
第二是对端口资源的占用，一个 TCP 连接至少消耗一个本地端口；
第二个危害是会造成严重的后果的，要知道，端口资源也是有限的，一般可以开启的端口为 32768～61000，也可以通过如下参数设置指定：

net.ipv4.ip_local_port_range

如果服务端 TIME_WAIT 状态过多，占满了所有端口资源，则会导致无法创建新连接。

如何优化 TIME_WAIT？

这里给出优化 TIME-WAIT 的几个方式，都是有利有弊：

打开 net.ipv4.tcp_tw_reuse 和 net.ipv4.tcp_timestamps 选项；
net.ipv4.tcp_max_tw_buckets
程序中使用 SO_LINGER ，应用强制使用 RST 关闭。


方式一：net.ipv4.tcp_tw_reuse 和 tcp_timestamps

如下的 Linux 内核参数开启后，则可以复用处于 TIME_WAIT 的 socket 为新的连接所用。

net.ipv4.tcp_tw_reuse = 1

使用这个选项，还有一个前提，需要打开对 TCP 时间戳的支持，即

net.ipv4.tcp_timestamps=1（默认即为 1）

这个时间戳的字段是在 TCP 头部的「选项」里，用于记录 TCP 发送方的当前时间戳和从对端接收到的最新时间戳。

由于引入了时间戳，我们在前面提到的 2MSL 问题就不复存在了，因为重复的数据包会因为时间戳过期被自然丢弃。

温馨提醒：net.ipv4.tcp_tw_reuse要慎用，因为使用了它就必然要打开时间戳的支持 net.ipv4.tcp_timestamps，当客户端与服务端主机时间不同步时，客户端的发送的消息会被直接拒绝掉。小林在工作中就遇到过。。。排查了非常的久


方式二：net.ipv4.tcp_max_tw_buckets

这个值默认为 18000，当系统中处于 TIME_WAIT 的连接一旦超过这个值时，系统就会将所有的 TIME_WAIT 连接状态重置。

这个方法过于暴力，而且治标不治本，带来的问题远比解决的问题多，不推荐使用。

方式三：程序中使用 SO_LINGER

我们可以通过设置 socket 选项，来设置调用 close 关闭连接行为。

structlingerso_linger;

so_linger.l_onoff = 1;

so_linger.l_linger = 0;

setsockopt(s, SOL_SOCKET, SO_LINGER, &so_linger, sizeof(so_linger));

如果l_onoff为非 0， 且l_linger值为 0，那么调用close后，会立该发送一个RST标志给对端，该 TCP 连接将跳过四次挥手，也就跳过了TIME_WAIT状态，直接关闭。

但这为跨越TIME_WAIT状态提供了一个可能，不过是一个非常危险的行为，不值得提倡。

如果已经建立了连接，但是客户端突然出现故障了怎么办？




在 Linux 内核可以有对应的参数可以设置保活时间、保活探测的次数、保活探测的时间间隔，以下都为默认值：

net.ipv4.tcp_keepalive_time=7200

net.ipv4.tcp_keepalive_intvl=75

net.ipv4.tcp_keepalive_probes=9

tcp_keepalive_time=7200：表示保活时间是 7200 秒（2小时），也就 2 小时内如果没有任何连接相关的活动，则会启动保活机制

tcp_keepalive_intvl=75：表示每次检测间隔 75 秒；
tcp_keepalive_probes=9：表示检测 9 次无响应，认为对方是不可达的，从而中断本次的连接。



listen 时候参数 backlog 的意义？

Linux内核中会维护两个队列：

未完成连接队列（SYN 队列）：接收到一个 SYN 建立连接请求，处于 SYN_RCVD 状态；
已完成连接队列（Accpet 队列）：已完成 TCP 三次握手过程，处于 ESTABLISHED 状态；

intlisten( intsocketfd, intbacklog)

参数一 socketfd 为 socketfd 文件描述符

参数二 backlog，这参数在历史有一定的变化
在早期 Linux 内核 backlog 是 SYN 队列大小，也就是未完成的队列大小。

在 Linux 内核 2.2 之后，backlog 变成 accept 队列，也就是已完成连接建立的队列长度，所以现在通常认为 backlog 是 accept 队列。


客户端调用 close，表明客户端没有数据需要发送了，则此时会向服务端发送 FIN 报文，进入 FIN_WAIT_1 状态；
服务端接收到了 FIN 报文，TCP 协议栈会为 FIN 包插入一个文件结束符 EOF 到接收缓冲区中，应用程序可以通过 read 调用来感知这个 FIN 包。这个 EOF 会被放在已排队等候的其他已接收的数据之后，这就意味着服务端需要处理这种异常情况，因为 EOF 表示在该连接上再无额外数据到达。此时，服务端进入 CLOSE_WAIT 状态；
接着，当处理完数据后，自然就会读到 EOF，于是也调用 close 关闭它的套接字，这会使得会发出一个 FIN 包，之后处于 LAST_ACK 状态；
客户端接收到服务端的 FIN 包，并发送 ACK 确认包给服务端，此时客户端将进入 TIME_WAIT 状态；
服务端收到 ACK 确认包后，就进入了最后的 CLOSE 状态；
客户端进过 2MSL 时间之后，也进入 CLOSED 状态；


##再深谈TCP/IP三步握手&四步挥手原理及衍生问题—长文解剖IP
https://www.zhoulujun.cn/html/theory/ComputerScienceTechnology/network/2015_0708_65.html

###初始化连接的 SYN 超时问题
Client发送SYN包给Server后挂了，Server回给Client的SYN-ACK一直没收到Client的ACK确认，这个时候这个连接既没建立起来，也不能算失败。这就需要一个超时时间让Server将这个连接断开，否则这个连接就会一直占用Server的SYN连接队列中的一个位置，大量这样的连接就会将Server的SYN连接队列耗尽，让正常的连接无法得到处理。

目前，Linux下默认会进行5次重发SYN-ACK包，重试的间隔时间从1s开始，下次的重试间隔时间是前一次的双倍，5次的重试时间间隔为1s, 2s, 4s, 8s, 16s，总共31s，第5次发出后还要等32s都知道第5次也超时了，所以，总共需要 1s + 2s + 4s+ 8s+ 16s + 32s = 63s，TCP才会把断开这个连接。由于，SYN超时需要63秒，那么就给攻击者一个攻击服务器的机会，攻击者在短时间内发送大量的SYN包给Server(俗称 SYN flood 攻击)，用于耗尽Server的SYN队列。对于应对SYN 过多的问题，linux提供了几个TCP参数：tcp_syncookies、tcp_synack_retries、tcp_max_syn_backlog、tcp_abort_on_overflow 来调整应对。

什么是 SYN 攻击（SYN Flood）

SYN 攻击指的是，攻击客户端在短时间内伪造大量不存在的IP地址，向服务器不断地发送SYN包，服务器回复确认包，并等待客户的确认。由于源地址是不存在的，服务器需要不断的重发直至超时，这些伪造的SYN包将长时间占用未连接队列，正常的SYN请求被丢弃，导致目标系统运行缓慢，严重者会引起网络堵塞甚至系统瘫痪。

SYN 攻击是一种典型的 DoS(Denial of Service)/DDoS(:Distributed Denial of Service) 攻击。

如何检测 SYN 攻击？

检测 SYN 攻击非常的方便，当你在服务器上看到大量的半连接状态时，特别是源IP地址是随机的，基本上可以断定这是一次SYN攻击。在 Linux/Unix 上可以使用系统自带的 netstats 命令来检测 SYN 攻击。

如何防御 SYN 攻击？

SYN攻击不能完全被阻止，除非将TCP协议重新设计。我们所做的是尽可能的减轻SYN攻击的危害，常见的防御 SYN 攻击的方法有如下几种：

缩短超时（SYN Timeout）时间
增加最大半连接数
过滤网关防护
SYN cookies技术

### Four-way Handshake 四次挥手
现来看下TCP各种状态含义解析(节选改编自《TCP、UDP 的区别，三次握手、四次挥手》

FIN_WAIT_1 ：这个状态得好好解释一下，其实FIN_WAIT_1 和FIN_WAIT_2 两种状态的真正含义都是表示等待对方的FIN报文。而这两种状态的区别是：- FIN_WAIT_1状态实际上是当SOCKET在ESTABLISHED状态时，它想主动关闭连接，向对方发送了FIN报文，此时该SOCKET进入到FIN_WAIT_1 状态。而当对方回应ACK报文后，则进入到FIN_WAIT_2 状态。当然在实际的正常情况下，无论对方处于任何种情况下，都应该马上回应ACK报文，所以FIN_WAIT_1 状态一般是比较难见到的，而FIN_WAIT_2 状态有时仍可以用netstat看到。

FIN_WAIT_2 ：上面已经解释了这种状态的由来，实际上FIN_WAIT_2状态下的SOCKET表示半连接，即有一方调用close()主动要求关闭连接。注意：FIN_WAIT_2 是没有超时的（不像TIME_WAIT 状态），这种状态下如果对方不关闭（不配合完成4次挥手过程），那这个 FIN_WAIT_2 状态将一直保持到系统重启，越来越多的FIN_WAIT_2 状态会导致内核crash。

TIME_WAIT ：表示收到了对方的FIN报文，并发送出了ACK报文。 TIME_WAIT状态下的TCP连接会等待2*MSL（Max Segment Lifetime，最大分段生存期，指一个TCP报文在Internet上的最长生存时间。每个具体的TCP协议实现都必须选择一个确定的MSL值，RFC 1122建议是2分钟，但BSD传统实现采用了30秒，Linux可以cat /proc/sys/net/ipv4/tcp_fin_timeout看到本机的这个值），然后即可回到CLOSED 可用状态了。如果FIN_WAIT_1状态下，收到了对方同时带FIN标志和ACK标志的报文时，可以直接进入到TIME_WAIT状态，而无须经过FIN_WAIT_2状态。

CLOSING ：这种状态在实际情况中应该很少见，属于一种比较罕见的例外状态。正常情况下，当一方发送FIN报文后，按理来说是应该先收到（或同时收到）对方的ACK报文，再收到对方的FIN报文。但是CLOSING 状态表示一方发送FIN报文后，并没有收到对方的ACK报文，反而却也收到了对方的FIN报文。什么情况下会出现此种情况呢？那就是当双方几乎在同时close()一个SOCKET的话，就出现了双方同时发送FIN报文的情况，这是就会出现CLOSING 状态，表示双方都正在关闭SOCKET连接。

CLOSE_WAIT ：表示正在等待关闭。怎么理解呢？当对方close()一个SOCKET后发送FIN报文给自己，你的系统毫无疑问地将会回应一个ACK报文给对方，此时TCP连接则进入到CLOSE_WAIT状态。接下来呢，你需要检查自己是否还有数据要发送给对方，如果没有的话，那你也就可以close()这个SOCKET并发送FIN报文给对方，即关闭自己到对方这个方向的连接。有数据的话则看程序的策略，继续发送或丢弃。简单地说，当你处于CLOSE_WAIT 状态下，需要完成的事情是等待你去关闭连接。

LAST_ACK ：当被动关闭的一方在发送FIN报文后，等待对方的ACK报文的时候，就处于LAST_ACK 状态。当收到对方的ACK报文后，也就可以进入到CLOSED 可用状态了。

Screen Shot 2018-11-05 at 20.55.01.jpg

TCP 的 Peer 两端同时断开连接
由上面的”TCP协议状态机 “图可以看出

TCP的Peer端在收到对端的FIN包前 发出了FIN包，那么该Peer的状态就变成了FIN_WAIT1

Peer在FIN_WAIT1状态下收到对端Peer对自己FIN包的ACK包的话，那么Peer状态就变成FIN_WAIT2，

Peer在FIN_WAIT2下收到对端Peer的FIN包，在确认已经收到了对端Peer全部的Data数据包后，就响应一个ACK给对端Peer，然后自己进入TIME_WAIT状态。

但是如果Peer在FIN_WAIT1状态下首先收到对端Peer的FIN包的话，那么该Peer在确认已经收到了对端Peer全部的Data数据包后，就响应一个ACK给对端Peer，然后自己进入CLOSEING状态，Peer在CLOSEING状态下收到自己的FIN包的ACK包的话，那么就进入TIME WAIT 状态。于是

TCP的Peer两端同时发起FIN包进行断开连接，那么两端Peer可能出现完全一样的状态转移 FIN_WAIT1——>CLOSEING——->TIME_WAIT，也就会Client和Server最后同时进入TIME_WAIT状态

TCP 的 TIME_WAIT 状态
要说明TIME_WAIT的问题，需要解答以下几个问题:

Peer两端，哪一端会进入TIME_WAIT呢？为什么?

相信大家都知道，TCP主动关闭连接的那一方会最后进入TIME_WAIT。

那么怎么界定主动关闭方呢？

是否主动关闭是由FIN包的先后决定的，就是在自己没收到对端Peer的FIN包之前自己发出了FIN包，那么自己就是主动关闭连接的那一方。对于TCP 的 Peer 两端同时断开连接 描述的情况，那么Peer两边都是主动关闭的一方，两边都会进入TIME_WAIT。为什么是主动关闭的一方进行TIME_WAIT呢，被动关闭的进入TIME_WAIT可以不呢？

我们来看看TCP四次挥手可以简单分为下面三个过程

过程一.主动关闭方 发送FIN；

过程二.被动关闭方 收到主动关闭方的FIN后 发送该FIN的ACK，被动关闭方发送FIN；

过程三.主动关闭方 收到被动关闭方的FIN后发送该FIN的ACK，被动关闭方等待自己FIN的ACK问题就在过程三中，据TCP协议规范，不对ACK进行ACK。

如果主动关闭方不进入TIME_WAIT，那么主动关闭方在发送完ACK就走了的话：如果最后发送的ACK在路由过程中丢掉了，最后没能到被动关闭方，这个时候被动关闭方 没收到自己FIN的ACK就不能关闭连接，接着被动关闭方 会超时重发FIN包，但是这个时候已经没有对端会给该FIN回ACK，被动关闭方就无法正常关闭连接了，所以主动关闭方需要进入TIME_WAIT 以便能够重发丢掉的被动关闭方FIN的ACK。

TIME_WAIT状态为什么需要经过2MSL的时间才关闭连接呢？

为了保证A发送的最后一个确认报文段能够到达B。这个确认报文段可能会丢失，如果B收不到这个确认报文段，其会重传第三次“挥手”发送的FIN+ACK报文，而A则会在2MSL时间内收到这个重传的报文段，每次A收到这个重传报文段后，就会重启2MSL计时器。这样可以保证A和B都能正常关闭连接。

为了防止已失效的报文段出现在下一次连接中。A经过2MSL时间后，可以保证在本次连接中传输的报文段都在网络中消失，这样一来就能保证在后面的连接中不会出现旧的连接产生的报文段了。

TIME_WAIT状态是用来解决或避免什么问题呢？

TIME_WAIT主要是用来解决以下几个问题：

上面解释为什么主动关闭方需要进入TIME_WAIT状态中提到的： 主动关闭方需要进入TIME_WAIT 以便能够重发丢掉的被动关闭方FIN的ACK。如果主动关闭方不进入TIME_WAIT，那么在主动关闭方对被动关闭方FIN包的ACK丢失了的时候，被动关闭方由于没收到自己FIN的ACK，会进行重传FIN包，这个FIN包到主动关闭方后，由于这个连接已经不存在于主动关闭方了，这个时候主动关闭方无法识别这个FIN包，协议栈会认为对方疯了，都还没建立连接你给我来个FIN包？于是回复一个RST包给被动关闭方，被动关闭方就会收到一个错误(我们见的比较多的：connect reset by peer，这里顺便说下 Broken pipe，在收到RST包的时候，还往这个连接写数据，就会收到 Broken pipe错误了)，原本应该正常关闭的连接，给我来个错误，很难让人接受。

防止已经断开的连接1中在链路中残留的FIN包终止掉新的连接2(重用了连接1的所有的5元素(源IP，目的IP，TCP，源端口，目的端口)），这个概率比较低，因为涉及到一个匹配问题，迟到的FIN分段的序列号必须落在连接2的一方的期望序列号范围之内，虽然概率低，但是确实可能发生，因为初始序列号都是随机产生的，并且这个序列号是32位的，会回绕。

防止链路上已经关闭的连接的残余数据包(a lost duplicate packet or a wandering duplicate packet) 干扰正常的数据包，造成数据流的不正常。这个问题和2）类似

TIME_WAIT会带来哪些问题呢

TIME_WAIT带来的问题注意是源于：一个连接进入TIME_WAIT状态后需要等待2*MSL(一般是1到4分钟)那么长的时间才能断开连接释放连接占用的资源，会造成以下问题

作为服务器，短时间内关闭了大量的Client连接，就会造成服务器上出现大量的TIME_WAIT连接，占据大量的tuple，严重消耗着服务器的资源。

作为客户端，短时间内大量的短连接，会大量消耗的Client机器的端口，毕竟端口只有65535个，端口被耗尽了，后续就无法在发起新的连接了。

( 由于上面两个问题，作为客户端需要连本机的一个服务的时候，首选UNIX域套接字而不是TCP )

TIME_WAIT很令人头疼，很多问题是由TIME_WAIT造成的，但是TIME_WAIT又不是多余的不能简单将TIME_WAIT去掉，那么怎么来解决或缓解TIME_WAIT问题呢？可以进行TIME_WAIT的快速回收和重用来缓解TIME_WAIT的问题。

有没一些清掉TIME_WAIT的技巧呢？

修改tcp_max_tw_buckets：tcp_max_tw_buckets 控制并发的TIME_WAIT的数量，默认值是180000。如果超过默认值，内核会把多的TIME_WAIT连接清掉，然后在日志里打一个警告。官网文档说这个选项只是为了阻止一些简单的DoS攻击，平常不要人为的降低它。

利用RST包从外部清掉TIME_WAIT链接：根据TCP规范，收到任何的发送到未侦听端口、已经关闭的连接的数据包、连接处于任何非同步状态（LISTEN, SYS-SENT, SYN-RECEIVED）并且收到的包的ACK在窗口外，或者安全层不匹配，都要回执以RST响应(而收到滑动窗口外的序列号的数据包，都要丢弃这个数据包，并回复一个ACK包)，内核收到RST将会产生一个错误并终止该连接。我们可以利用RST包来终止掉处于TIME_WAIT状态的连接，其实这就是所谓的RST攻击了。为了描述方便：假设Client和Server有个连接Connect1，Server主动关闭连接并进入了TIME_WAIT状态，我们来描述一下怎么从外部使得Server的处于 TIME_WAIT状态的连接Connect1提前终止掉。要实现这个RST攻击，首先我们要知道Client在Connect1中的端口port1(一般这个端口是随机的，比较难猜到，这也是RST攻击较难的一个点)，利用IP_TRANSPARENT这个socket选项，它可以bind不属于本地的地址，因此可以从任意机器绑定Client地址以及端口port1，然后向Server发起一个连接，Server收到了窗口外的包于是响应一个ACK，这个ACK包会路由到Client处，这个时候99%的可能Client已经释放连接connect1了，这个时候Client收到这个ACK包，会发送一个RST包，server收到RST包然后就释放连接connect1提前终止TIME_WAIT状态了。提前终止TIME_WAIT状态是可能会带来(问题二、)中说的三点危害，具体的危害情况可以看下RFC1337。RFC1337中建议，不要用RST过早的结束TIME_WAIT状态。

 TCP 的 TIME_WAIT 状态
要说明TIME_WAIT的问题，需要解答以下几个问题:

Peer两端，哪一端会进入TIME_WAIT呢？为什么?

相信大家都知道，TCP主动关闭连接的那一方会最后进入TIME_WAIT。

那么怎么界定主动关闭方呢？

是否主动关闭是由FIN包的先后决定的，就是在自己没收到对端Peer的FIN包之前自己发出了FIN包，那么自己就是主动关闭连接的那一方。对于TCP 的 Peer 两端同时断开连接 描述的情况，那么Peer两边都是主动关闭的一方，两边都会进入TIME_WAIT。为什么是主动关闭的一方进行TIME_WAIT呢，被动关闭的进入TIME_WAIT可以不呢？

我们来看看TCP四次挥手可以简单分为下面三个过程

过程一.主动关闭方 发送FIN；

过程二.被动关闭方 收到主动关闭方的FIN后 发送该FIN的ACK，被动关闭方发送FIN；

过程三.主动关闭方 收到被动关闭方的FIN后发送该FIN的ACK，被动关闭方等待自己FIN的ACK问题就在过程三中，据TCP协议规范，不对ACK进行ACK。

如果主动关闭方不进入TIME_WAIT，那么主动关闭方在发送完ACK就走了的话：如果最后发送的ACK在路由过程中丢掉了，最后没能到被动关闭方，这个时候被动关闭方 没收到自己FIN的ACK就不能关闭连接，接着被动关闭方 会超时重发FIN包，但是这个时候已经没有对端会给该FIN回ACK，被动关闭方就无法正常关闭连接了，所以主动关闭方需要进入TIME_WAIT 以便能够重发丢掉的被动关闭方FIN的ACK。

TIME_WAIT状态为什么需要经过2MSL的时间才关闭连接呢？

为了保证A发送的最后一个确认报文段能够到达B。这个确认报文段可能会丢失，如果B收不到这个确认报文段，其会重传第三次“挥手”发送的FIN+ACK报文，而A则会在2MSL时间内收到这个重传的报文段，每次A收到这个重传报文段后，就会重启2MSL计时器。这样可以保证A和B都能正常关闭连接。

为了防止已失效的报文段出现在下一次连接中。A经过2MSL时间后，可以保证在本次连接中传输的报文段都在网络中消失，这样一来就能保证在后面的连接中不会出现旧的连接产生的报文段了。

TIME_WAIT状态是用来解决或避免什么问题呢？

TIME_WAIT主要是用来解决以下几个问题：

上面解释为什么主动关闭方需要进入TIME_WAIT状态中提到的： 主动关闭方需要进入TIME_WAIT 以便能够重发丢掉的被动关闭方FIN的ACK。如果主动关闭方不进入TIME_WAIT，那么在主动关闭方对被动关闭方FIN包的ACK丢失了的时候，被动关闭方由于没收到自己FIN的ACK，会进行重传FIN包，这个FIN包到主动关闭方后，由于这个连接已经不存在于主动关闭方了，这个时候主动关闭方无法识别这个FIN包，协议栈会认为对方疯了，都还没建立连接你给我来个FIN包？于是回复一个RST包给被动关闭方，被动关闭方就会收到一个错误(我们见的比较多的：connect reset by peer，这里顺便说下 Broken pipe，在收到RST包的时候，还往这个连接写数据，就会收到 Broken pipe错误了)，原本应该正常关闭的连接，给我来个错误，很难让人接受。

防止已经断开的连接1中在链路中残留的FIN包终止掉新的连接2(重用了连接1的所有的5元素(源IP，目的IP，TCP，源端口，目的端口)），这个概率比较低，因为涉及到一个匹配问题，迟到的FIN分段的序列号必须落在连接2的一方的期望序列号范围之内，虽然概率低，但是确实可能发生，因为初始序列号都是随机产生的，并且这个序列号是32位的，会回绕。

防止链路上已经关闭的连接的残余数据包(a lost duplicate packet or a wandering duplicate packet) 干扰正常的数据包，造成数据流的不正常。这个问题和2）类似

TIME_WAIT会带来哪些问题呢

TIME_WAIT带来的问题注意是源于：一个连接进入TIME_WAIT状态后需要等待2*MSL(一般是1到4分钟)那么长的时间才能断开连接释放连接占用的资源，会造成以下问题

作为服务器，短时间内关闭了大量的Client连接，就会造成服务器上出现大量的TIME_WAIT连接，占据大量的tuple，严重消耗着服务器的资源。

作为客户端，短时间内大量的短连接，会大量消耗的Client机器的端口，毕竟端口只有65535个，端口被耗尽了，后续就无法在发起新的连接了。

( 由于上面两个问题，作为客户端需要连本机的一个服务的时候，首选UNIX域套接字而不是TCP )

TIME_WAIT很令人头疼，很多问题是由TIME_WAIT造成的，但是TIME_WAIT又不是多余的不能简单将TIME_WAIT去掉，那么怎么来解决或缓解TIME_WAIT问题呢？可以进行TIME_WAIT的快速回收和重用来缓解TIME_WAIT的问题。

有没一些清掉TIME_WAIT的技巧呢？

修改tcp_max_tw_buckets：tcp_max_tw_buckets 控制并发的TIME_WAIT的数量，默认值是180000。如果超过默认值，内核会把多的TIME_WAIT连接清掉，然后在日志里打一个警告。官网文档说这个选项只是为了阻止一些简单的DoS攻击，平常不要人为的降低它。

利用RST包从外部清掉TIME_WAIT链接：根据TCP规范，收到任何的发送到未侦听端口、已经关闭的连接的数据包、连接处于任何非同步状态（LISTEN, SYS-SENT, SYN-RECEIVED）并且收到的包的ACK在窗口外，或者安全层不匹配，都要回执以RST响应(而收到滑动窗口外的序列号的数据包，都要丢弃这个数据包，并回复一个ACK包)，内核收到RST将会产生一个错误并终止该连接。我们可以利用RST包来终止掉处于TIME_WAIT状态的连接，其实这就是所谓的RST攻击了。
为了描述方便：假设Client和Server有个连接Connect1，Server主动关闭连接并进入了TIME_WAIT状态，我们来描述一下怎么从外部使得Server的处于 TIME_WAIT状态的连接Connect1提前终止掉。要实现这个RST攻击，首先我们要知道Client在Connect1中的端口port1(一般这个端口是随机的，比较难猜到，这也是RST攻击较难的一个点)，利用IP_TRANSPARENT这个socket选项，它可以bind不属于本地的地址，因此可以从任意机器绑定Client地址以及端口port1，然后向Server发起一个连接，Server收到了窗口外的包于是响应一个ACK，这个ACK包会路由到Client处，这个时候99%的可能Client已经释放连接connect1了，这个时候Client收到这个ACK包，会发送一个RST包，server收到RST包然后就释放连接connect1提前终止TIME_WAIT状态了。提前终止TIME_WAIT状态是可能会带来(问题二、)中说的三点危害，具体的危害情况可以看下RFC1337。RFC1337中建议，不要用RST过早的结束TIME_WAIT状态。


#高性能网络编程
##高性能网络编程7--tcp连接的内存使用
https://cloud.tencent.com/developer/article/1345061

##大并发下TCP内存消耗优化小记（86万并发业务正常服务） 
https://www.cnblogs.com/x_wukong/p/7998903.html

TCP能够使用的内存：这三个值就是TCP使用内存的大小，单位是页，每个页是4K的大小，如下：
 
这三个值分别代表
Low：6179424   （6179424*4/1024/1024大概23g）
Pressure：8239232 （8239232*4/1024/1024大概31g）
High：12358848   （echo 12358848*4/1024/1024大概47g）

这个也是系统装后的默认取值，也就是说最大有47个g（75%的内存）可以用作TCP连接，这三个量也同时代表了三个阀值，TCP的使用小于第二个值时kernel不会有任何提示操作，当大于第二个值时进入压力模式，当高于第三个值时将不接受新的TCP连接，同时会报出“Out  of  socket memory”或者“TCP:too many of orphaned sockets”。

TCP读缓存大小，单位是字节：第一个是最小值4K，第二个是默认值85K，第三个是最大值16M

##tcp内存占用/socket内存占用 
https://www.cnblogs.com/shengulong/p/11623621.html

##Linux 中每个 TCP 连接最少占用多少内存？
https://zhuanlan.zhihu.com/p/25241630
https://www.ctolib.com/topics-110215.html

