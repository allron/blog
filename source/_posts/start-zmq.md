---
title: Start 0MQ
date: 2017-12-08 13:54:24
tags: zqm
---

# What is ZMQ


ZeroMQ zero-em-queue, ØMQ:
 Ø  Connect your code in any language, on any platform.
 Ø  Carries messages across inproc, IPC, TCP, TIPC, multicast.
 Ø  Smart patterns like pub-sub, push-pull, and router-dealer.
 Ø  High-speed asynchronous I/O engines, in a tiny library.
 Ø  Backed by a large and active open source community.
 Ø  Supports every modern language and platform.
 Ø  Build any architecture: centralized, distributed, small, or large.
 Ø  Free software with full commercial support.


# How to install

get source form (zeromq.org 4.2.2)[https://github.com/zeromq/libzmq/releases/download/v4.2.2/zeromq-4.2.2.tar.gz]

```sh
tar xvzf zeromq-4.2.2.tar.gz
cd zeromq-4.2.2
./configure
make -j4
sudo make install
```

# Frist Demo
zmq_inproc - 0MQ local in-process (inter-thread) communication transport

```c
// init zmq ctx
void *zmq_ctx=NULL;
zmq_ctx = zmq_ctx_new();

 void *bindSocket = zmq_socket(ctx, ZMQ_PULL);
 assert (bindSocket);

//  Assign the in-process name "my-endpoint"
rc = zmq_bind(bindSocket, "inproc://first-endpoint");
assert (rc == 0);

 void *connectSocket = zmq_socket(zmq_ctx, ZMQ_PAIR);
 assert (connectSocket);
 int rc = zmq_connect(connectSocket, ZMQ_ADDR_INIPC);

// send data
 zmq_send(connectSocket, buf, len, 0);

//revice data
 zmq_msg_t msg;
 rc = zmq_msg_init(&msg);
 assert (rc == 0);
 rc = zmq_msg_recv(&msg, bindSocket, 0);
 void *data = zmq_msg_data(&msg);
 size_t len = zmq_msg_size(&msg);
```


