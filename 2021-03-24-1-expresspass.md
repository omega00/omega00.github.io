## Credit-Scheduled Delay-Bounded Congestion Control for Datacenters  Expresspass

## Introduction

1. buffer per port per Gbps getting smaller

2. 最直观的解决办法   直接分配带宽给每一条流

   接收端主动反馈credit包，在发生拥塞之前，发送端即可获得网络的状态   可以有效地控制Incast  credit包可以反向的控制数据包的发送

3. 设计思想：（1）在交换机处进行credit包的速率限制  （2） 对称哈希以实现路径同步  （3）反馈credit包的控制  （4）随机抖动  （5）网络最大队列的测量

## Motivation

1. 通过测试   一个server持续发送200B给worker  每一个worker回复1000B  理想的速率控制以及DCTCP效果不好  而使用credit进行控制则可以很好的控制
2. 有界的队列堆积     当一个credit包到达发送端之后，发送端变会发送一个数据包，而在这个机制下，队列只会在以下情况堆积 （1）路径长度的不同  （2）终端处理包速度的不同
3. 快速收敛问题    快速收敛和低的缓存占用是相矛盾的  存在超出缓冲区以及丢包的风险

## Design

Design 挑战

​			（a）图中的链路2没有被公平的分享    flow0的速率是其他之和    （b）图中  链路1没有被完全占用   对于只有两条流的情况，flow0的credit经过link 1之后只剩下百分之五十，随后又会减小到总体的33%
<img src="https://raw.githubusercontent.com/omega00/picsave/master/image-20210324163825317-1620831177776.png?token=AHHQMHIZH4EACH53Y4QMINLATPWAU" alt="image-20210324163825317" style="zoom:67%;" />
<img src="https://raw.githubusercontent.com/omega00/picsave/master/image-20210324175015466.png?token=AHHQMHNI7PCIOFO5XJOFFO3ATPVK4" alt="image-20210324175015466" style="zoom:67%;" />

1. 在交换机处确保零丢包   交换机处的最大队列被限制

   主机的处理时延   credit buffer的大小  以及  链路速率  影响了需要的缓存大小

2. 在端侧确保公平的  credit drop

   在端侧发送credit包的时候加入一定的抖动    避免在同时发送credit包时产生问题

3. credit包反馈机制

   credit包的反馈速率是动态进行调整的  当有credit包出现丢包时，说明网络发生拥塞、

4. credit反馈的开始以及结束

   通过  SYN包在三次握手之后开始传输   对于UDP 在第一个RTT内发送 credit

   在流结束的时候，发送credit stop



## 反馈机制的不足之处：

​		短的流会导致大量的credit包被浪费   这很大程度的影响了长流的流完成时间      有一个解决办法是使用不同优先级来区分包  

​		现有的反馈机制假设所有的终端有着相同的链路速率