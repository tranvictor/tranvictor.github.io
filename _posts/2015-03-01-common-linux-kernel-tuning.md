---
layout: post
title: "Some common linux kernel tuning for Linux server"
date: 2015-03-01
tags: linux
---

Using AWS EC2 Linux server grants you full control of the server, however you usually have to tune the kernel to make the server able to handle many internet connections and open files. This is some common tunning based on *Bryan Veal and Annie Foong (Performance Scalability of a Multi-Core Server, Nov 2007, page 4/10)*

* Add all the following lines to the file: “/etc/sysctl.conf”

```
  fs.file-max = 5000000
  net.core.netdev_max_backlog = 400000
  net.core.optmem_max = 10000000
  net.core.rmem_default = 10000000
  net.core.rmem_max = 10000000
  net.core.somaxconn = 100000
  net.core.wmem_default = 10000000
  net.core.wmem_max = 10000000
  net.ipv4.conf.all.rp_filter = 1
  net.ipv4.conf.default.rp_filter = 1
  net.ipv4.tcp_congestion_control = bic
  net.ipv4.tcp_ecn = 0
  net.ipv4.tcp_max_syn_backlog = 12000
  net.ipv4.tcp_max_tw_buckets = 2000000
  net.ipv4.tcp_mem = 30000000 30000000 30000000
  net.ipv4.tcp_rmem = 30000000 30000000 30000000
  net.ipv4.tcp_sack = 1
  net.ipv4.tcp_syncookies = 0
  net.ipv4.tcp_timestamps = 1
  net.ipv4.tcp_wmem = 30000000 30000000 30000000
  #
  # Optionally, avoid TIME_WAIT states on localhost no-HTTP Keep-Alive tests:
  # “error: connect() failed: Cannot assign requested address (99)”
  # On Linux, the 2MSL time is hardcoded to 60 seconds in /include/net/tcp.h:
  # define TCP_TIMEWAIT_LEN (60*HZ). This option is safe to use in production.
  #
  net.ipv4.tcp_tw_reuse = 1
  #
  # WARNING:
  # ——–
  # The option below lets you reduce TIME_WAITs by several orders of magnitude
  # but this option is for benchmarks, NOT for production servers (NAT issues)
  # So, uncomment the line below if you know what you’re doing.
  #
  #net.ipv4.tcp_tw_recycle = 1
  #
  net.ipv4.ip_local_port_range = 1024 65535
  net.ipv4.ip_forward = 0
  net.ipv4.tcp_dsack = 0
  net.ipv4.tcp_fack = 0
  net.ipv4.tcp_fin_timeout = 30
  net.ipv4.tcp_orphan_retries = 0
  net.ipv4.tcp_keepalive_time = 120
  net.ipv4.tcp_keepalive_probes = 3
  net.ipv4.tcp_keepalive_intvl = 10
  net.ipv4.tcp_retries2 = 15
  net.ipv4.tcp_retries1 = 3
  net.ipv4.tcp_synack_retries = 5
  net.ipv4.tcp_syn_retries = 5
  net.ipv4.tcp_moderate_rcvbuf = 1
  kernel.sysrq = 0
  kernel.shmmax = 67108864
```

* Then add also the 2 following lines to the file: /etc/secutity/limits.conf

```
* soft nofile 1000000
* hard nofile 1000000
```
