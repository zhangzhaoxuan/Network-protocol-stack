# Network-protocol-stack

## 1 协议帧及路由过程

### 1.1 概述

| 帧名   | 介绍   |
| ---- | ---- |
| RREQ | 路由请求 |
| RREP | 路由应答 |
| RERR | 路由错误 |

### 1.2 RREQ帧

   (_当源节点S需要向目的节点D发送数据包时，但有没有目的节点D的路由入口时，会发送RREQ帧，RREQ帧会广播。_)

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

    -帧中字段的含义
      type : 1 (默认1)
      J : join
      R : repair
      G : Gratuitous RREP (免费路由回复标记)  说明是否应该向目标节点发送一个路由回复的报文
      D : 仅允许目的节点进行回复
      U : 指示目标节点序列号未知
      Reserved : 填充0 会被接收端忽视的字段
      Hop Count : 从发起节点到处理该节点请求的跳数
      RREQ ID : ID
      Destination IP Address : 目的节点IP地址
      Destination Sequence Number : 目标节点序列号 发起节点在以前能通向目的节点的节点中找到的最新的序列号
      Originator IP Address : 发起本条路由请求信息的IP地址
      Originator Sequence Number : 发起者的路由表中现在正在使用的序列号

### 1.3 RREP帧

  (_当RREQ到达时，目的节点发送的反向路由帧，使用这个帧来使网络上的各个节点建立前一个节点的路由，节点只对收到的第一次RREQ产生反应，重复发送的RREQ将不能产生重复的RREP帧_)

     -帧的格式
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |     Type      |R|A|    Reserved     |Prefix Sz|   Hop Count   |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                     Destination IP address                    |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                  Destination Sequence Number                  |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                    Originator IP address                      |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                           Lifetime                            |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

    -帧中字段的含义
      type : 2 (默认2)
      R : repair
      A : acknowledgment required (需要确认)
      Reserved : 填充0 会被接收端忽视的字段
      Prefix SZ : 前缀长度 值为 0 - 31 前缀指的是 IPV4地址 - 主机地址的长度
      Hop Count : 从发起节点到处理该节点请求的跳数
      Destination IP Address : 目的节点IP地址
      Destination Sequence Number : 目标节点序列号 从路由表中查找对应的序列号
      Originator IP Address : 发起本条路由请求信息的IP地址
      lifetime 生命时长 在规定的这段时间中 接受到这条消息的节点将认为这条路径是可用的

### 1.4RERR帧

  (_路由错误帧，用来进行路由的错误控制_)

       -帧的格式
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |     Type      |N|          Reserved           |   DestCount   |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |            Unreachable Destination IP Address (1)             |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |         Unreachable Destination Sequence Number (1)           |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-|
    |  Additional Unreachable Destination IP Addresses (if needed)  |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |Additional Unreachable Destination Sequence Numbers (if needed)|
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

      -帧中字段的含义
        type : 3 (默认3)
        N : No delete 当上一个节点对这个连接做了本地修复，就将这个标志位置真，也即不用删除这个路由
        Reserved : 填充0 会被接收端忽视的字段    
        DestCount : 本消息内包含的不可达目的节点的数目
        Unreachable Destination IP Address : 不能到的节点目的节点IP地址
        Unreachable Destination Sequence Number : 与上面的目标节点对应的序列号
        Additional : 可以包含多个不可达节点，用来表示多个不可达节点有一个 加一对儿消息

### 1.5 RREP-ACK 帧

  (_收到需要确认的帧时用来回复的帧_)

    -帧的格式
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |     Type      |   Reserved    |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

    -帧中字段的含义
    type : 4(默认4)
    Reserved  : 填充0 会被接收端忽视的字段 .

## 2 路由表

    - 目的节点 ip 地址
    - 目的节点序号
    - 目的节点序列号是否正确的标记
    - 其他状态和路由标记
    - 跳数
    - 下一跳
    - 先驱表
    - 生命（路由过期或者应当删除的时间）

## 3 代码介绍

### 3.1 文件介绍

| 文件              | 说明                 |
| --------------- | ------------------ |
| list.h          | 定义链表结构             |
| routing_table.h | 路由表定义              |
| aodv_hello.c    | hello 消息           |
| aodv_neighbor.c | 添加邻居节点并且处理邻居节点断开事件 |
| aodv_rerr.c     | rerr 消息的定义发送及处理    |
| aodv_rrep.c     | rrep 消息的定义发送及处理    |
| aodv_rreq.c     | rreq 消息的定义发送及处理    |
| aodv_socket.c   | 处理aodv的socket套接字   |
| aodv_timeout.c  | 超时处理               |

### 3.2 全局变量

| 变量  | 数据类型 | 说明  |
| --- | ---- | --- |

### 3.3 main 函数

  main函数可以简单地分成三个部分

-   while之前的代码 进行了一系列定义和初始化工作。

        ```C
        static char *ifname = NULL;	/* Name of interface to attach to */
        fd_set rfds,sigaction readers;
        int n, nfds = 0, i;
        int daemonize = 0;
        struct timeval *timeout;
        struct timespec timeout_spec;
        struct  sigact;
        sigset_t mask, origmask;

        /* Remember the name of the executable... */
        progname = strrchr(argv[0], '/');

        if (progname)
        progname++;
        else
        progname = argv[0];
        debug = 1;
          memset (&sigact, 0, sizeof(struct sigaction));
        sigact.sa_handler = signal_handler;

        /* This server should shut down on these signals. */
        sigaction(SIGTERM, &sigact, 0);
        sigaction(SIGHUP, &sigact, 0);
        sigaction(SIGINT, &sigact, 0);

        sigaddset(&mask, SIGTERM);
        sigaddset(&mask, SIGHUP);
        sigaddset(&mask, SIGINT);
        /* Only capture segmentation faults when we are not debugging... */
        #ifndef DEBUG
        sigaddset(&mask, SIGSEGV);
        #endif

        /* Block the signals we are watching here so that we can
         * handle them in pselect instead. */
        sigprocmask(SIG_BLOCK, &mask, &origmask);

          ```


-   进入while
        这个while实际上是个永真循环，在这部分

        协议将根据用户提供的不同的参数对变量进行不同的设置留给后面的操作使用，下表是对应关系

      | 参数名    | 功能       |
      | ----- | -------- |
      | -d  | 守护进程模式     |
      | -g  | 在每个RREQ消息上强制设置gratuitous标记     |
      | -h  | 打印所有参数及含义     |
      | -i | 表示要绑定的接口 |
      | -j | 触发hello-jitter功能，默认开启|
      | -o | opt-hellos 设置只在转发数据包的时候发送HELLO消息|
      | -l | log输出日志 |
      | -r | log-rt-table每隔一段时间记录路由表 |
      | -n | 接到n个HELLO之后才当作邻居 |
      | -u | 侦测并避免单向链路 |
      | -w | 开启实验性的因特网网关支持 |
      | -x | 禁用RREQ消息的扩展环搜索法 |
      | -D | 启用重启延迟的等待 |
      | -L | 开启本地修复 |
      | -f | 开启链路层反馈 |
      | -R | 开启RREQ和RRER消息的速率限制 |
      | -q | 为控制包设置一个信号质量最小阈值 |
      | -V | 输出版本信息 |

      ```C
        int opt;
        opt = getopt_long(argc, argv, "i:fjln:dghoq:r:s:uwxDLRV", longopts, 0);
        if (opt == EOF)
        break;
        switch (opt) {
        case 0:
        break;
        case 'd':
        debug = 0;
        daemonize = 1;
        break;
        case 'f':
        llfeedback = 1;
        active_route_timeout = ACTIVE_ROUTE_TIMEOUT_LLF;
        break;
        case 'g':
        rreq_gratuitous = !rreq_gratuitous;
        break;
        case 'i':
        ifname = optarg;
        break;
        case 'j':
        hello_jittering = !hello_jittering;
        break;
        case 'l':
        log_to_file = !log_to_file;
        break;
        case 'n':
        if (optarg && isdigit(*optarg)) {
        receive_n_hellos = atoi(optarg);
        if (receive_n_hellos < 2) {
          fprintf(stderr, "-n should be at least 2!\n");
          exit(-1);
        }
        }
        break;
        case 'o':
        optimized_hellos = !optimized_hellos;
        break;
        case 'q':
        if (optarg && isdigit(*optarg))
        qual_threshold = atoi(optarg);
        break;
        case 'r':
        if (optarg && isdigit(*optarg))
        rt_log_interval = atof(optarg) * 1000;
        break;
        case 'u':
        unidir_hack = !unidir_hack;
        break;
        case 'w':
        internet_gw_mode = !internet_gw_mode;
        break;
        case 'x':
        expanding_ring_search = !expanding_ring_search;
        break;
        case 'L':
        local_repair = !local_repair;
        break;
        case 'D':
        wait_on_reboot = !wait_on_reboot;
        break;
        case 'R':
        ratelimit = !ratelimit;
        break;
        case 'V':
        printf
        ("\nAODV-UU v%s, %s � Uppsala University & Ericsson AB.\nAuthor: Erik Nordstr�m, <erik.nordstrom@it.uu.se>\n\n",
        AODV_UU_VERSION, DRAFT_VERSION);
        exit(0);
        break;
        case '?':
        case ':':
        exit(0);
        default:
        usage(0);
        }
        }
      ```

-   后面代码仍然在while中\
    分别执行了

      - 检查是否是root启动
          ```C
          if (geteuid() != 0) {
            fprintf(stderr, "must be root\n");
            exit(1);
            }

          ```
      - 检查是否-d启动
          ```C
          if (daemonize) {
           if (fork() != 0)
               exit(0);
           /* Close stdin, stdout and stderr... */
           /*  close(0); */
           close(1);
           close(2);
           setsid();
           }
           /*
           1. 如果是-d用作守护进程 将关闭标准输入输出流 并执行setsid()函数 当进程是会话的领头进程时setsid()调用失败并返回（-1）。
           setsid()调用成功后，返回新的会话的ID，调用setsid函数的进程成为新的会话的领头进程，并与其父进程的会话组和进程组脱离。
           由于会话对控制终端的独占性，进程同时与控制终端脱离。
           2. 如果是父进程直接退出
           */

           ```

      - 初始化数据结构和服务(_数据包输入输出队列_)

          ```C

          /* Initialize data structures and services... */
          rt_table_init();
          log_init();
          /*   packet_queue_init(); */
          host_init(ifname);
          /*   packet_input_init(); */
          nl_init();
          nl_send_conf_msg();
          aodv_socket_init();

      #ifdef LLFEEDBACK
          if (llfeedback) {
          llf_init();
          }
      #endif

          ```

      - 设置socket套接字备用

          ```C
          /* Set sockets to watch... */
          FD_ZERO(&readers);
          for (i = 0; i < nr_callbacks; i++) {
          FD_SET(callbacks[i].fd, &readers);
          if (callbacks[i].fd >= nfds)
              nfds = callbacks[i].fd + 1;
          }

          ```
      - 重启定时器 reboot

          ```C
            if (wait_on_reboot) {
          timer_init(&worb_timer, wait_on_reboot_timeout, &wait_on_reboot);
          timer_set_timeout(&worb_timer, DELETE_PERIOD);
          alog(LOG_NOTICE, 0, __FUNCTION__,
               "In wait on reboot for %d milliseconds. Disable with \"-D\".",
               DELETE_PERIOD);
          }

          ```

      - 准备好第一个HELLO帧、初始化第一个路由表

          ```C
              if (!optimized_hellos && !llfeedback)
    	hello_start();

        if (rt_log_interval)
    	log_rt_table_init();

        ```

      - 定时器处理

        ```C
          while (1) {
    	memcpy((char *) &rfds, (char *) &readers, sizeof(rfds));

    	timeout = timer_age_queue();

    	timeout_spec.tv_sec = timeout->tv_sec;
    	timeout_spec.tv_nsec = timeout->tv_usec * 1000;

    	if ((n = pselect(nfds, &rfds, NULL, NULL, &timeout_spec, &origmask)) < 0) {
    	    if (errno != EINTR)
    		alog(LOG_WARNING, errno, __FUNCTION__,
    		     "Failed select (main loop)");
    	    continue;
    	}

    	if (n > 0) {
    	    for (i = 0; i < nr_callbacks; i++) {
    		if (FD_ISSET(callbacks[i].fd, &rfds)) {
    		    /* We don't want any timer SIGALRM's while executing the
    		       callback functions, therefore we block the timer... */
    		    (*callbacks[i].func) (callbacks[i].fd);
    		}
    	    }
    	}
        }

      ```

  ps：

  1. pselect()用于IO复用，它们监视多个文件描述符的集合，判断是否有符合条件的时间发生。
  2. FD_ISSET() 检查参数1是否在这个参数2里面 在这个函数中如果检查成功在执行回调函数处理对应事件


main 函数中的其他函数

  函数名 | 作用
  - | -
  clearup() | 清空套接字、数据表等数据结构和模块
  usage(int status) | 打印出所有命令行参数和其作用
  set_kernel_options | 设置内核选项
  find_default_gw() | 寻找默认网关
  get_if_info(char *ifname , int type) | 根据名字和类型,寻找相应的接口,返回其信息
  attach_callback_func(int fd , callback_func_t func) | 增加一个callback数据结构元素,包括一个描述符和一个函数
  load_modules() | 加载某个模块
  remove_modules() | 删除某个模块
  host_init | 初始化某个端口
  signal_handler(int type) | 信号处理器,根据信号种类导向不同的处理
