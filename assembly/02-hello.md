```assembly
mov ax, 3
int 0x10 ; 将显示模式设置成文本模式

mov ax, 0xb800 ; 将0xb800放到ax寄存器中,此时ax寄存器中保存了0xb800
mov ds, ax ; 将ax寄存器中的值放到ds段寄存器中，此时段寄存器ds中存放了0xb800

mov byte [0], 'H' ; 显示字符式两个字节；第一个字节表示字符，第二个字节表示样式 mov byte [1], 11110000b H会闪烁显示
mov byte [2], 'e'
mov byte [4], 'l'
mov byte [6], 'l'
mov byte [8], 'o'
mov byte [10], ','
mov byte [12], 'W'
mov byte [14], 'o'
mov byte [16], 'r'
mov byte [18], 'l'
mov byte [20], 'd'

halt:
    jmp halt

times 510 - ($ - $$) db 0
db 0x55, 0xaa
```

- 8086的寻址方式： 段寄存器 * 16  + 地址 (0xb800 * 0x10 + 0 = 0xb8000)

- 实模式下的内存区域表

| 起始地址  | 结束地址  | 大小     | 用途               |
| --------- | --------- | -------- | ------------------ |
| `0x000`   | `0x3FF`   | 1KB      | 中断向量表         |
| `0x400`   | `0x4FF`   | 256B     | BIOS数据区         |
| `0x500`   | `0x7BFF`  | 29.75KB  | 可用区域           |
| `0x7C00`  | `0x7DFF`  | 512B     | MBR加载区域        |
| `0x7E00`  | `0x9FBFF` | 607.6KB  | 可用区域           |
| `0x9FC00` | `0x9FFFF` | 1KB      | 扩展BIOS数据区     |
| `0xA0000` | `0xAFFFF` | 64KB     | 用于彩色显示适配区 |
| `0xB0000` | `0xB7FFF` | 32KB     | 用于黑白显示适配区 |
| `0xB8000` | `0xBFFFF` | 32KB     | 用于文本显示适配区 |
| `0xC0000` | `0xC7FFF` | 32KB     | 显示适配器BIOS     |
| `0xC8000` | `0xEFFFF` | 160KB    | 映射内存           |
| `0xF0000` | `0xFFFEF` | 64KB-16B | 系统BIOS           |
| `0xFFFF0` | `0xFFFFF` | 16B      | 系统BIOS入口地址   |

