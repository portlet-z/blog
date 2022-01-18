## 需要掌握的知识

1. 使用命令行进行网络分析的优势
2. 利用命令行对捕获文件进行调优
3. 掌握命令行工具常用的命令
4. 掌握命令行工具与第三方辅助工具的使用方法

- tshark -r file.pcapng | grep -e GET ： -r读取， 查看file.pcapng数据包，过滤GET关键字
- capinfos file.pcapng: 查看数据包的答题信息