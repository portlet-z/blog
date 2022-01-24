## ABNF(扩充巴科斯-瑙尔范式)操作符

- 空白字符：用来分隔定义中的各个元素
- 选择 / : 表示多个规则都是可供选择的规则
  - start-line = request-line / status-line
- 值范围 %c##-##:
  - OCTAL = "0" / "1" / "2" / "3" / "4" / "5" / "6" / "7" 与 OCTAL = %x30-37 等价
- 序列组合() : 将规则组合起来，视为单个元素
- 不定量重复m * n
  - 元素表示零个或更多元素：*(header-field CRLF)
  - 1* 元素表示一个或更多元素， 2*4元素表示两个至四个元素
- 可选序列[]
  - [message-body]



## ABNF(扩充巴克斯-瑙尔范式)核心规则

| 规则   | 形式定义                                  | 意义                                   |
| ------ | ----------------------------------------- | -------------------------------------- |
| ALPHA  | %x41-5A / %x61-7A                         | 大写和小写ASCII字母(A-Z a-z)           |
| DIGIT  | %x30-39                                   | 数字(0-9)                              |
| HEXDIG | DIGIT / "A" / "B" / "C" / "D" / "E" / "F" | 十六进制数字(0-9, A-F, a-f)            |
| DQUOTE | %22                                       | 双引号                                 |
| SP     | %20                                       | 空格                                   |
| HTAB   | %09                                       | 横向制表符                             |
| WSP    | SP / HTAB                                 | 空格或横向制表符                       |
| LWSP   | *(WSP / CRLF WSP)                         | 直线空白（晚于换行）                   |
| VCHAR  | %x21-7E                                   | 可见（打印）字符                       |
| CHAR   | %x01-7F                                   | 任何7位US-ASCII字符串，不包括NUL(%x00) |
| OCTET  | %x00-FF                                   | 8位数据                                |
| CTL    | %x00-1F / %x7F                            | 控制字符                               |
| CR     | %x0D                                      | 回车                                   |
| LF     | %x0A                                      | 换行                                   |
| CRLF   | CR LF                                     | 互联网标准换行                         |
| BIT    | "0" / "1"                                 | 二进制数字                             |



## 基于ABNF 描述的HTTP协议格式

HTTP-message = start-line *(header-field CRLF) CRLF [message-body]

- start-line = request-line / status-line
  - request-line = method SP request-target SP HTTP-version CRLF
  - status-line = HTTP-version SP status-code SP reason-phrase CRLF
- header-field = field-name ":" OWS field-value OWS
  - OWS = *(SP / HTAB)
  - field-name = token
  - field-value = *(field-content / obs-fold)
- message-body = *OCTET