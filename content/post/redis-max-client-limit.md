+++
categories = ["Redis"]
date = "2017-03-30T22:50:49+08:00"
tags = ["Redis"]
title = "记一次redis连接数超限的事故"

+++

最近出了一次事故，应用crash，并报错: max number of clients reached. 查了一下reids的连接：
```
netstat -anp |grep 6379 | wc -l
```
发现，连接数达10000多个，大致扫了一下，发现某个ip的连接占了7000多个。于是去把这个ip的服务器上的应用都停止掉了，可是连接数还是没降下来。
研究了一段时间，发现redis有client list和client kill的命令，想到，可以通过这种方式kill掉这个ip的client。
但是进去redis里面执行client list和client kill命令，居然报错：max number of clients reached.

中途有想过用tcpkill之类的工具，最后没采用这种方法。

情急之下，想到了取巧的办法：

* 先用config set maxclients 20000将最大连接数变成20000
* 然后执行client list命令，发现不会报错了，那就是说明可以执行client kill了
* kill掉这些ip的客户端
```
sudo netstat -anp |grep 6379| grep 'xxx.xxx.xxx.xxx' | awk '{print $5}' | xargs -I % redis-cli -p 6379 client kill %
```
等tcp四次招手结束后，连接数终于降下来了
