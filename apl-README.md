抗丢包
====================================================================================================

在网络上进行实时音视频通信时，传输层通常使用 UDP 协议，相比 TCP 它能提供更高的吞吐量和更低的延迟。但
是 UDP 是无链接的、不可靠的数据报协议，经常出现乱序、包重复、丢包等问题。

对于乱序和包重复的问题，解决办法很简单。发送方在每个发送的数据包中加入一个递增的序列号或时间戳，接收
方根据序列号对数据包排序即可（通常使用 RTP 协议)。

对于丢包的问题就稍麻烦点，发送方使用 [FEC](http://openfec.org) 对已发送了的 n 个数据包编码生成 k 个
冗余包并将这 k 个数据包以同样的方式发送出去，总共发送了 (n + k) 个数据包。接收方只要接收到这 (n + k)
个数据包中的任意 n 个，就可以使用 FEC 解码恢复丢失的数据包。

n 的值意味着延迟，例如第一个数据包丢失了，接收方至少需等到第 n + 1 个数据包才能恢复这个丢失的包。
k 的值意味着带宽，k 越大冗余包占的比率就越大，同时抗丢包能力就越好。通常设置 k / (n + k) 略大于实际
的丢包率。

避免连续的发送数据包，保证两次发送之间有一个时间间隔，也可以减少丢包的概率。

<!-- vim:set tw=100 ts=4 sw=4 et: -->
