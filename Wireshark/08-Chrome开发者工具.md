- 快捷键Command+Option+I(Mac), Ctrl+Alt+I(Windows)

- NetWork面板

  - 控制器：控制面板的外观与功能
    - 抓包（红心）
    - 停止抓包
    - 清除请求
    - 要跨页面加载保存请求（Preserve log）
    - 屏幕截图（Capture screenshots）
    - 重新执行XHR请求：右键点击请求选择Replay XHR
    - 停用浏览器缓存
    - 手动清除浏览器缓存：右键点击请求选择Clear Browser Cache
    - 离线模拟：Offline
    - 模拟慢速网络连接：Network Throttling,可自定义网速
    - 手动清除Cookie:右键点击请求选择Clear Browser Cookies
    - 隐藏Filters窗格
    - 隐藏Overview窗格
  - 过滤器：过滤请求列表中显示的资源
    - 按住Command(Mac)或Ctrl(Window/Linux),然后点击过滤器可以同时选择多个过滤器
    - 按类型过滤
      1. XHR, JS, CSS, Img, Media, Font, Doc, WS(WebSocket),Manifest或Other
      2. 多类型，按住Command(Mac)或Ctrl(Windows)
      3. 按时间过滤：概览面板，拖动滚动条
      4. 隐藏Data URLs: CSS图片等小文件以BASE64格式嵌入HTML中，以减少HTTP请求次数
    - 按属性过滤
      1. domain: 仅显示来自指定域的资源，*多个
      2. has-response-header:显示包含指定HTTP响应头的资源
      3. is: 使用is:running可以查找WebSocket资源， is:from-cache可查找缓存读出的资源
      4. larger-than: 显示大于指定大小的资源
      5. method: 显示指定HTTP方法类型检索的资源
      6. mime-type: 显示指定MIME类型的资源
      7. 多属性间通过AND操作
      8. mixed-content: 显示所有混合内容资源(mixed-content:all),或者仅显示当前显示的资源(mixed-content:displayed)
      9. scheme: 显示通过未保护HTTP(scheme:http)或受保护HTTPS(scheme:https)检索的资源
      10. set-cookie-domina: 显示具有Set-Cookie标头并且Domain属性与指定值匹配的资源
      11. set-cookie-name:显示具有Set-Cookie标头并且名称与指定值匹配的资源
      12. set-cookie-value: 显示具有Set-Cookie标头并且值与指定值匹配的资源
      13. status-code:仅显示HTTP状态代码与指定代码匹配的资源
  - 概览：显示HTTP请求，响应的时间轴
  - 请求列表：默认时间排序，可选择显示列
    - 时间排序，默认
    - 按列排序
    - 按活动时间排序：
      1. Start Time: 发出的第一个请求位于顶部
      2. Response Time: 开始下载的第一个请求位于顶部
      3. End Time: 完成的第一个请求位于顶部
      4. Total Duration: 连接设置时间和请求、响应时间最短位于请求顶部
      5. Latency: 等待最短响应时间的请求位于顶部
    - 请求列表
      1. Name:资源的名称
      2. Status: HTTP状态码
      3. Type: 请求资源的MIME类型
      4. Initiator:发起请求的对象或进程。它可能有以下几种值
         - Parser(解析器)： Chrome的HTML解析器发起了请求。鼠标悬停显示JS脚本
         - Redirect(重定向)：HTTP重定向启动了请求
         - Script(脚本)：脚本启动了请求
         - Other(其他)：一些其他进程或动作发起请求，例如用户点击链接跳转到页面或在地址栏中输入网址
      5. Size: 服务器返回的响应大学
      6. Time: 总持续时间，从请求的开始到接收响应中的最后一个字节
      7. Waterfall:各请求相关活动的直观分析图
  - 概要：请求总数，总数据量，总花费时间等
    - 查看头部
    - 查看Cookie
    - 预览响应正文：查看图像用
    - 查看响应正文
    - 时间详细分步
    - 导出数据为HAR格式
    - 查看未压缩的资源大小： Use Larger Request Rows
    - 浏览器加载时间(概要，概览，请求列表)

  