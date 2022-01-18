## 实验目的

1. 熟练使用Wireshark的图形显示功能来辅助分析
2. 学会灵活使用图形界面的筛选器，对不同的网络情况进行对比分析
3. 知道IO Graphs, Round Trip Time Graph以及Flow Graph的区别，学会依据不同的目的选择图表进行分析

- Statistics -> I/O Graphs 可以查看数据包的IO可视化视图
- tcp.analysis.flags && !tcp.analysis.windows.update
- tcp.analysis.duplicate_ack
- tcp.analysis.lost_segment
- tcp.analysis.retransmission
- Statistics -> TCP Stream Graphs -> Round Trip Time 查看数据包RTT往返时间
- Statistics -> Flow Graph 查询数据包的流向图