# Network protocol stack
## 2.1 功能模块分析
### AODV协议是通过不断广播hello报文维护路由状态的协议，一旦发现某一条链路断开，就发送ERROR信息，并在路由表中移除这条信息
***
## 2.2 hello报文模块
###  一个节点可以通过广播本地的Hello消息提供链接信息。如果一个节点是主动路由的一部分，它应该只使用Hello消息。每经过HELLO——INTERVAL毫秒，节点检查它在过去的HELLO_INTERVAL是否发出了广播，如果没有，则发出TTL=1 的RREP消息
|  全局变量   | 数据类型 | 说明 |
|  ----    | ----  | --- |
|  undir_hack   | int |  |
|   单元格  | 单元格 |
