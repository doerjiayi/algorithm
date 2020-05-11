
#linux 指令

##head tail
https://blog.csdn.net/signjing/article/details/69357769
https://zhidao.baidu.com/question/430426385.html

##tee
https://blog.csdn.net/weixin_34362790/article/details/86443986

##tcp dump
https://www.cnblogs.com/f-ck-need-u/p/7064286.html

##awk sed
https://www.cnblogs.com/-zyj/p/5763303.html

##find
find grep 联合使用 过滤所有子目录、文件
find . -type f -name '*.log*' | xargs grep --color -n 'GetConnectIdentify'

##traceroute
https://www.cnblogs.com/ftl1012/p/traceroute.html
traceroute -n -m 5 -q 4 -w 3 www.baidu.com 
说明： -n 显示IP地址，不查主机名，  -m 设置跳数   
         -q 4每个网关发送4个数据包    -w 把对外发探测包的等待响应时间设置为3秒