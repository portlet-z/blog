## 评估Web架构的关键属性

1. 性能Performance:影响高可用的关键因素
   - 网络性能 Network Performance
     - Throughput 吞吐量：小于等于带宽
     - Overhead开销：首次开销，每次开销
   - 用户感知到的性能User-perceived Performance
     - Latency延迟：发起请求到接收到响应的时间
     - Completion完成时间：完成一个应用动作所花费的时间
   - 网络效率Network Efficiency
     - 重用缓存，减小交互次数，数据传输距离更近，COD
2. 可伸缩性Scalability: 支持部署可以互相交互的大量组件
3. 简单性Simplicity: 易理解，易实现，易验证
4. 可见性Visiable: 对两个组件间的交互进行监视或者仲裁的能力。如缓存，分层设计等
5. 可移植性Portability: 在不同的环境下运行的能力
6. 可靠性Reliablity: 出现部分故障时，对整体影响的程度
7. 可修改性Modifiability:对系统作出修改的难易程度，由可进化性，可定制性，可扩展性，可配置性，可重用性构成
   - 可进化性Evolvability:一个组件独立升级而不影响其他组件
   - 可扩展性Extensibility:向系统添加功能，而不会影响到系统的其他部分
   - 可定制性Customizability:临时性，定制性地更改某一要素来提供服务，不对常规客户产生影响
   - 可配置性Configurability:应用部署后可通过修改配置提供新的功能
   - 可重用性Reusability:组件可以不做修改在其他应用再使用



## 5种架构风格

1. 数据流风格Data-flow Styles
   - 优点：简单性，可进化性，可扩展性，可配置性，可重用性
   - 管道与过滤器Pipe And Filter, PF
     - 每个Filter都有输入端和输出端，只能从输入端读取数据，处理后再从输出端产生数据
   - 统一接口的管道与过滤器Uniform Pipe And Filter, UPF
     - 在PF上增加了统一接口的约束，所有Filter过滤器必须具备同样的接口
2. 复制风格 Replication Styles
   - 优点：用户可察觉的性能，可伸缩性，网络效率，可靠性也可以得到提升
   - 复制仓库Replicated Repository, RR
     - 多个进程提供相同的服务，通过反向代理对外提供集中服务
   - 缓存$
     - RR的变体，通过复制请求的结果，为后续请求复用
3. 分层风格Hierachical Styles
   - 优点：简单性，可进化性，可伸缩性
   - 客户端服务器Client-Server, CS
   - 分层系统Layered System, LS
   - 分层客户端服务器 Layered Client-Server, LCS
   - 无状态，客户端服务器Client-Stateless-Server CSS
   - 缓存，无状态，客户端服务器 Client-Cache-Stateless-Server C$SS
   - 分层，缓存，无状态，客户端服务器Layered-Client-Cache-Stateless-Server, LC$SS
   - 远程会话Remote Session, RS
   - 远程数据访问Remote Data Access, RDA
4. 移动代码风格Mobile Code Styles
   - 优点：可移植性，可扩展性，网络效率
   - 虚拟机Virtual Machine, VM
   - 远程求职Remote Evaluation, REV
   - 按需代码Code on Demand, COD
   - 移动代码Mobile Agent , MA
5. 点对点风格Peer-to-Peer Styles
   - 优点：可进化性，可重用性，可扩展性，可配置性