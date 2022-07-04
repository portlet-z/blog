## Java对象头

以32位虚拟机为例

普通对象

<table style="text-align: center">
  <tr>
    <td colspan="2">Object Header(64 bits)</td>
  </tr>
  <tr>
    <td>Mark Word (32 bits)</td>
    <td>Klass Word (32 bits)</td>
  </tr>
</table>