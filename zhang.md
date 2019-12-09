# Network protocol stack
## 2.1 功能模块分析
### AODV协议是通过不断广播hello报文维护路由状态的协议，一旦发现某一条链路断开，就发送ERROR信息，并在路由表中移除这条信息
***
## 2.2 hello报文模块
###  一个节点可以通过广播本地的Hello消息提供链接信息。如果一个节点是主动路由的一部分，它应该只使用Hello消息。每经过HELLO_INTERVAL毫秒，节点检查它在过去的HELLO_INTERVAL是否发出了广播，如果没有，则发出TTL=1 的RREP消息
|  消息种类  | 说明 | 用途 |  
|  ----    | ----  | --- |
|  RREQ  | 路由请求  | 当某结点接收到新的目标节点时广播REEQ消息 
|  RREP  | 路由回复 | 当一个节点向给它发送RREQ的节点单播一条RREQ时，原节点将相邻节点状态信息存入路由表 |
|  RERR  | 路由错误 | 若活动路由表某一条连接发生断裂，REER将指出不能到达的节点或子网

## 1.3 AODV路由表分析
### 1.3.1 路由表结构分析
源代码：
```C++
struct rt_table {
    list_t l;
    struct in_addr dest_addr;    /* IP address of the destination */
    u_int32_t dest_seqno;
    unsigned int ifindex;    /* Network interface index... */
    struct in_addr next_hop;    /* IP address of the next hop to the dest */
    u_int8_t hcnt;        /* Distance (in hops) to the destination */
    u_int16_t flags;        /* Routing flags */
    u_int8_t state;        /* The state of this entry */
    struct timer rt_timer;    /* The timer associated with this entry */
    struct timer ack_timer;    /* RREP_ack timer for this destination */
    struct timer hello_timer;
    struct timeval last_hello_time;
    u_int8_t hello_cnt;
    hash_value hash;
    int nprec;            /* Number of precursors */
    list_t precursors;        /* List of neighbors using the route */
};
```
#### 路由表信息
|  变量名称  |  数据类型  |    作用   |
|  ----    | ----  | --- |
| dest_addr(目的节点地址) | in_addr | 用于标志使用此路由的最终目的节点，决定了数据分组转发方向。 |
| dest_seqno(目的节点序列号) | u_int32_t | 反映此路由的新鲜度，一般序列号越大路由越新鲜， 这是保证开环的重要措施，在路由发现和路由应答更新路由时需要进行序列号的比较。 |
| flags(路由状态标志) | u_int16_t(无效、有效、正在修复等) | 反映此路由目前的状态，主要用于告知数据分组经过此节点的时候处理方式。如果路  由处于无效状态，那么数据分组将丢失；如果处于修复状态，那么数据分组进入等待路由队列中；如果有效状态，那么直接转发。 |
| next_hop(下一跳的ip地址) | in_addr | 据分组经过本节点之后，数据分组将被直接转发的中继节点，通常下一跳节点应该出现在当前节点的邻节点列表中。 |
| hcnt(到达目的ip的跳数) | u_int8_t | 到达目的ip所需经历的子网的数量 |
| precursors(前驱节点表) | list_t |  使用这条路由的所有直接前驱节点列表。在出现断链的时候可以通过前驱节点列表中是否存有节点而决定是否广播RERR消息 |
| rt_timer(路由生命周期) | timer | 路由有效的生命期，在数据分组转发使用当前路由时会更新路由的有效生命期，当较长时间不使用此路由时，此路由的有效期将会过期，在路由管理时将会使路由失效 |


### 1.3.2 AODV路由维护
#### hello报文路由维护
1.节点周期性发送TTL=1的hello报文
```C++
 if (optimized_hellos &&
	timeval_diff(&now, &this_host.fwd_time) > ACTIVE_ROUTE_TIMEOUT) {
	hello_stop();
	return;
    }

    time_diff = timeval_diff(&now, &this_host.bcast_time);
    jitter = hello_jitter();

    /* This check will ensure we don't send unnecessary hello msgs, in case
       we have sent other bcast msgs within HELLO_INTERVAL */
    if (time_diff >= HELLO_INTERVAL) {

	for (i = 0; i < MAX_NR_INTERFACES; i++) {
	    if (!DEV_NR(i).enabled)
		continue;
#ifdef DEBUG_HELLO
	    DEBUG(LOG_DEBUG, 0, "sending Hello to 255.255.255.255");
#endif
	    rrep = rrep_create(flags, 0, 0, DEV_NR(i).ipaddr,
			       this_host.seqno,
			       DEV_NR(i).ipaddr,
			       ALLOWED_HELLO_LOSS * HELLO_INTERVAL);

	    /* Assemble a RREP extension which contain our neighbor set... */
	    if (unidir_hack) {
		int i;

		if (ext)
		    ext = AODV_EXT_NEXT(ext);
		else
		    ext = (AODV_ext *) ((char *) rrep + RREP_SIZE);

		ext->type = RREP_HELLO_NEIGHBOR_SET_EXT;
		ext->length = 0;

		for (i = 0; i < RT_TABLESIZE; i++) {
		    list_t *pos;
		    list_foreach(pos, &rt_tbl.tbl[i]) {
			rt_table_t *rt = (rt_table_t *) pos;
			/* If an entry has an active hello timer, we assume
			   that we are receiving hello messages from that
			   node... */
			if (rt->hello_timer.used) {
#ifdef DEBUG_HELLO
			    DEBUG(LOG_INFO, 0,
				  "Adding %s to hello neighbor set ext",
				  ip_to_str(rt->dest_addr));
#endif
			    memcpy(AODV_EXT_DATA(ext), &rt->dest_addr,
				   sizeof(struct in_addr));
			    ext->length += sizeof(struct in_addr);
			}
		    }
		}
		if (ext->length)
		    msg_size = RREP_SIZE + AODV_EXT_SIZE(ext);
	    }
	    dest.s_addr = AODV_BROADCAST;
	    aodv_socket_send((AODV_msg *) rrep, dest, msg_size, 1, &DEV_NR(i));
	}

	timer_set_timeout(&hello_timer, HELLO_INTERVAL + jitter);
    } else {
	if (HELLO_INTERVAL - time_diff + jitter < 0)
	    timer_set_timeout(&hello_timer,
			      HELLO_INTERVAL - time_diff - jitter);
	else
	    timer_set_timeout(&hello_timer,
			      HELLO_INTERVAL - time_diff + jitter);
    }
}


```
节点周期性接收hello报文维护路由表中hello_timer选项

