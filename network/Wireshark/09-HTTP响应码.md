## 用于文档管理的WebDAV方法(RFC2518)

- PROPFIND: 从Web资源中检索以XML格式存储的属性。它也被重载，以允许一个检索远程系统的集合结构(也叫目录层次结构)
- PROPPATCH: 在单个原子性动作中更改和删除资源的多个属性
- MKCOL: 创建集合或者目录
- COPY: 将资源从一个URI复制到另一个URI
- MOVE: 将资源从一个URI移动到另一个URI
- LOCK:锁定一个资源。WebDAV支持共享锁和互斥锁
- UNLOCK: 解除资源的锁定



## 响应码1xx:请求已接收到，需要进一步处理才能完成，HTTP1.0不支持

- 100:Continue 上传大文件前使用
- 101 Switch Protocols: 协议升级使用
- 102 Processing: 



## 2xx: 成功处理请求

- 200 OK
- 201 Created
- 202 Accepted
- 203 Non-Authoritative Information
- 204 No Content
- 205 Reset Content
- 206 Partial Content
- 207 Multi-Status
- 208 Already Reported



## 3xx:重定向

- 300 Multiple Choices
- 301 Moved Permanently
- 302 Found
- 303 See Other
- 304 Not Modified
- 307 Temporary Redirect
- 308 Permanent Redirect



## 4xx客户端出现错误

- 400 Bad Request
- 401 Unauthorized
- 407 Proxy Authentication Required
- 403 Forbidden
- 404 Not Found
- 410 Gone
- 405 Method Not Allowed
- 406 Not Acceptable
- 408 Request Timeout
- 409 Conflict
- 411 Length Required