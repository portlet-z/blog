# pcap_findalldevs
## 描述
- 使libpcap(或Npcap)库中的一个函数，用于获取系统中所有可用的网络设备。它可以列出当前计算机上安装的网络接口信息，供用户选择一个网络设备进行数据包捕获
- pcap_freealldevs函数用于释放资源
## 函数原型
```c
int pcap_findalldevs(pcap_if_t** alldevs, char* errbuf);
```
## 参数说明
- `pcap_if_t** alldevs`
    - 类型：执行指针的指针，pcap_if_t使一个结构体，表示一个网络设备的信息
    - 描述：返回的指针指向一个链表，链表中的每个元素代表一个网络设备。链表的结束由NULL指针标识
    - 每个pcap_if_t结构体包含了网络设备的名称、描述、地址信息等。
- `char* errbuf`
    - 类型：字符数组（通常大小为`PCAP_ERRBUF_SIZE`）
    - 描述：存储错误信息的缓冲区。如果函数执行失败，错误信息将存储在该缓冲区中。
## 返回值
- 成功：返回0，并且alldevs将包含所有可用的网络设备链表。
- 失败：返回-1，并且可以通过errbuf获取错误信息
## `pcap_if_t`结构体
- pcap_if_t结构体用于存储网络设备的信息，它的定义如下：
```c
struct pcap_if {
	struct pcap_if *next; //指向下一个设备的指针（链表结构）
	char *name;		//设备的名称（如eth0, lo, \Device\NPF_{...}）
	char *description;	// 设备的描述（如"Intel(R) Ethernet"）
	struct pcap_addr *addresses; //设备的IP地址和掩码等信息
	bpf_u_int32/*int*/ flags;	//设备的标志，指示该设备的特性
};
```
- name: 设备的名称（如eth0, lo, \Device\NPF_{...}）
- description: 设备的描述（如"Intel(R) Ethernet"）
- address: 设备的IP地址和掩码等信息使用pcap_addr结构体存储
- flags: 设备的标志,表示设备是否支持混杂模式等特性。
- next: 指向下一个设备的指针（链表结构）
# pcap_open_live
## 描述
- 实时捕获：用于打开一个网络接口，从网络上实时捕获数据包
- 通常用于实时监听网卡上的网络流量
- pcap_close函数用于释放资源
## 函数原型
```c
pcap_t* pcap_open_live(const char* device, int snaplen, int promisc, int to_ms, char* errbuf);
```
## 参数说明
- device: 要捕获的网卡接口名称（如eth0, \Device\NPF_{...}）
- snaplen: 捕获数据包的最大长度（建议65535表示捕获整个数据包）
- promisc: 是否启用混杂模式(1为启用，0为禁用)。启用混杂模式，网卡会接收所有通过网卡的数据包，而不仅仅使目的地址为本机的
- to_ms: 读取超时时间（单位为毫秒）。指定多长时间后返回捕获的数据包，即使没有捕获到数据
- errbuf: 存储错误消息的缓冲区
## 返回值
- 成功时：返回一个pcap_t* 类型的句柄，用于后续的捕获操作
- 失败时：返回NULL, 错误信息存储在errbuf中
## 示例
```c
char errbuf[PCAP_ERRBUF_SIZE];
pcap_t* handle = pcap_open_live("eth0", 65535, 1, 1000, errbuf);
if (!handle) {
    fprintf(stderr, "Error opening device: %s\n", errbuf);
    return 1;
}
``` 
# pcap_open_offline
## 描述
- 离线分析：用于打开一个已经保存的捕获文件（如.pcap或.pcapng文件），并从文件中读取数据包
- 通常用于对先前捕获的数据包进行分析
- pcap_close函数用于释放资源
## 函数原型
```c
pcap_t* pcap_open_offline(const char* filename, char* errbuf);
```
## 参数说明
- filename: 要打开的捕获文件的路径
- errbuf: 存储错误消息的缓冲区
## 返回值
- 成功时：返回一个pcap_t* 类型的句柄，用于读取数据包
- 失败时：返回NULL, 错误信息存储在errbuf中。
## 示例
```c
char errbuf[PCAP_ERRBUF_SIZE];
pcap_t* handle = pcap_open_offline("capture.pcap", errbuf);
if (!handle) {
    fprintf(stderr, "Error opening file: %s\n", errbuf);
    return 1;
}
```
## pcap_open_live对比pcap_open_offline
|特性|pcap_open_live|pcap_open_offline|
|-|-|-|
|数据来源|网络接口的实时流量|捕获文件中的历史数据|
|用途|实时监控网络流量|离线分析捕获数据|
|输入参数|网卡接口名称|捕获文件路径|、
|是否需要网卡支持|是|否|
|是否需要管理员权限|是（捕获实时数据包通常需要管理员权限）|否|
|处理效率|实时捕获，可能丢包|离线读取，通常不会丢包|
# pcap_loop
## 描述
- Npcap和libpcap提供的一个函数，用于连续捕获网络数据包。
## 函数声明
```c
int pcap_loop(pcap_t* p, int cnt, pcap_handler callback, u_char* user);
```
## 参数说明
- `pcap_t* p`
    - 类型：指向打开的捕获会话的句柄
    - 描述：这是通过pcap_open_live或其他函数(pcap_open_offline)返回的指针，表示一个打开的捕获接口
```c
pcap_t* handle = pcap_open_live(device_name, 65535, 1, 1000, errbuf);
pcap_loop(handle, 10, packet_handler, NULL);
```
- `int cnt`
    - 类型：整数
    - 描述：捕获的数据包数量。
    - 正整数：指定捕获的最大数据包数。例如，传入10表示只捕获10个数据包
    - 0或负数：表示无限捕获，直到手动停止（例如，用户按下Ctrl+C或调用pcap_breakloop）
```c
//捕获5个数据包后停止
pcap_loop(handle, 5, packet_handler, NULL);
```
- `pcap_handler callback`
    - 类型：函数指针
    - 描述：这是一个回调函数，pcap_loop在捕获到每个数据包时调用它。
    - 函数原型: `packet_handler callback(u_char* user, const struct pcap_pkthdr* header, const u_char* pkt_data);`
    - u_char* user: 用户自定义数据（通过pcap_loop的第4个参数传递）
    - struct pcap_pkthdr* header:指向捕获数据包头部信息的结构体，包括时间戳、长度等信息
    - const u_char* pkt_data: 指向实际数据包内容的指针
```c
void packet_handler(unsigned char* user, const struct pcap_pkthdr* header, const u_char* pkt_data) {
    printf("Packet captured with length: %d\n", header->len);
}
```
- `u_char* user`
    - 类型：用户自定义的参数
    - 描述：这是一个用户自定义的数据指针，会被直接传递给回调函数的第一个参数u_char* user
    - 可以传递任意数据（例如，统计信息的结构体或其他上下文数据）
    - 如果不需要，可以传入NULL
```c
int packet_count = 0;
pcap_loop(handle, 10, packet_handler, (u_char*)&packet_count);
```
## 返回值
- 成功时：
    - 返回0表示捕获了指定数量的数据包
    - 返回-2表示捕获循环被中断（通过pcap_breakloop调用）
- 失败时：返回-1表示发生错误（可以通过pcap_geterr获取错误消息）
# demo
- nps.h
```c
#pragma once
#include "pcap.h"
void devices_info();
pcap_if_t* device_find(pcap_if_t* all_devices, const char* name);
void device_handler(unsigned char* user, const struct pcap_pkthdr* header, const unsigned char* pkt_data);
```
- device.c
```c
#include <stdio.h>
#include <stdlib.h>
#include "pcap.h"
#include <winsock2.h>

void devices_info() {
    pcap_if_t* all_devices;
    char eb[PCAP_ERRBUF_SIZE];
    if (pcap_findalldevs_ex(PCAP_SRC_IF_STRING, NULL, &all_devices, eb) == -1) {
        exit(1);
    }
    for (const pcap_if_t* device = all_devices; device != NULL; device = device->next) {
        printf("\nDevice Name: %s\n", device->name);
        if (device->description != NULL) {
            printf("Description: %s\n", device->description);
        }
        struct pcap_addr* addr;
        for (addr = device->addresses; addr != NULL; addr = addr->next) {
            if (addr->addr && addr->addr->sa_family == AF_INET) {
                struct sockaddr_in* ip_addr = (struct sockaddr_in*)addr->addr;
                struct sockaddr_in* netmask = (struct sockaddr_in*)addr->netmask;
                printf("IP Address: %s\n", inet_ntoa(ip_addr->sin_addr));
                if (netmask) {
                    printf("Subnet Mask: %s\n", inet_ntoa(netmask->sin_addr));
                }
            }
        }
    }
    pcap_freealldevs(all_devices);
}

pcap_if_t* device_find(pcap_if_t* all_devices, const char* name) {
    pcap_if_t* device;
    char errbuf[PCAP_ERRBUF_SIZE];
    char nbuf[64];
    sprintf(nbuf, "\\Device\\NPF_{%s", name);
    if (pcap_findalldevs(&all_devices, errbuf) == -1) {
        return NULL;
    }
    for (device = all_devices; device != NULL; device = device->next) {
        if (strcmp(device->name, nbuf) == 0) {
            return device;
        }
    }
    return NULL;
}
void device_handler(unsigned char* user, const struct pcap_pkthdr* header, const unsigned char* pkt_data) {
    printf("\nPacket captured:\n");
    printf("Timestamp: %ld.%ld seconds\n", header->ts.tv_sec, header->ts.tv_usec);
    printf("Packet length: %d bytes\n", header->len);
}
```
- nps.c
```c
#include "nps.h"
#include <stdio.h>
#include "pcap.h"

int main(void) {
    pcap_if_t* all_devices = NULL;
    char errbuf[PCAP_ERRBUF_SIZE];
    pcap_if_t* device = device_find(all_devices, "745C1BEE-9624-4588-817A-60AC4A23FF0B");
    if (device != NULL) {
        printf("%p\n", device);
    }
    //打开设备
    pcap_t* handle = pcap_open_live(device->name, 65536, 1, 655360, errbuf);
    if (!handle) {
        fprintf(stderr, "Couldn't open device: %s\n", errbuf);
        pcap_freealldevs(all_devices);
        return 1;
    }
    //开始抓包
    pcap_loop(handle, 5, device_handler, NULL);
    pcap_close(handle);
    pcap_freealldevs(all_devices);

    printf("Hello, World!\n");
    system("pause");
    return 0;
}
```