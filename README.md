# Network-protocol-stack

## 1 协议帧及路由过程
  ### 1.1 aodv 帧
  帧名 | 介绍
     -|-
     RREQ | 路由请求
     RREP | 路由应答
     RERR | 路由错误
     HELLO | 活跃路由链路检测

   2.1 RREQ帧
   (*当源节点S需要向目的节点D发送数据包时，但有没有目的节点D的路由入口时，会发送RREQ帧，RREQ帧会广播。*)


    -帧的格式
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |     Type      |J|R|G|D|U|   Reserved          |   Hop Count   |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                            RREQ ID                            |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                    Destination IP Address                     |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                  Destination Sequence Number                  |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                    Originator IP Address                      |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                  Originator Sequence Number                   |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+



## 2 代码介绍

  ###  文件介绍

  文件 | 说明
  -|-
  list.h | 定义链表结构
  routing_table.h | 路由表定义
  aodv_hello.c | hello 消息
  aodv_neighbor.c | 添加邻居节点并且处理邻居节点断开事件
  aodv_rerr.c | rerr 消息的定义发送及处理
  aodv_rrep.c | rrep 消息的定义发送及处理
  aodv_rreq.c | rreq 消息的定义发送及处理
  aodv_socket.c | 处理aodv的socket套接字
  aodv_timeout.c | 超时处理

  ### 2.2 全局变量

  变量 | 数据类型 | 说明
  -|-|-
