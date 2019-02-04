# network-performance-tools
Documentation on network tools useful for quick performance analysis by ordinary developers. 

1. [iperf3](#iperf3)
2. [dd-Netcat](#dd-Netcat)

## iperf3 

1. [iperf3](https://github.com/esnet/iperf)
* Traditional network tool used to examine network throughput performance between two servers. When run, it returns data transfered, bandwidth, packet loss and related statistics over a TCP connection. 
* Language - C
* Authors - Esnet @ Berkeley

### Install

```Bash
$ sudo apt install iperf3	#Debian/Ubuntu
$ sudo yum install iperf3	#RHEL/CentOS
$ sudo dnf install iperf3	#Fedora 22+ 
```

### Usage

1. Start an iperf3 server in the suspected machine 

```Bash
iperf3 -s
```

2. Connect to the server FROM your laptop or some other client and observe network performance statistics

```Bash
iperf3 -c <server ip>
```

![iperf](https://github.com/peterlamar/network-performance-tools/blob/master/img/iperf3.png)

### Pros
* Mature tool, been around since 2008

### Cons 
* Can be difficult to install on older servers which do not support iperf dependencies in thier kernel Centos < 7

### Reference
* Advanced [usage](https://www.tecmint.com/test-network-throughput-in-linux/)
* Man [page](https://fasterdata.es.net/performance-testing/network-troubleshooting-tools/iperf/)


## dd-Netcat 

2. dd & Netcat
* Copies data between two servers and measures data transfered


### Install

dd typically comes installed in Linux

```Bash
$ sudo apt install netcat	#Debian/Ubuntu
$ sudo yum install netcat	#RHEL/CentOS
$ sudo dnf install netcat	#Fedora 22+ 
```

### Usage

1. Start an netcat server in the suspected machine 

```Bash
nc -v -l 2222 > /dev/null
```

2. Connect to the server FROM your laptop or some other client and observe network performance statistics

```Bash
dd if=/dev/zero bs=1024K count=512 | nc -v <server ip> 2222
```

3. Observe file transfered. I had to interrupt transfer with Control C due to slow speed

```Bash
27262976 bytes (27 MB, 26 MiB) copied, 53.4426 s, 510 kB/s
```

### Pros
* More likely to be installed already

### Cons 
* Limited in functionality, truly basic test. 

### Reference
* Difference between GNU and BSD [netcat](https://www.quora.com/What-is-the-difference-between-the-openBSD-netcat-and-the-GNU-netcat)
* Linux IO with [dd](https://www.thomas-krenn.com/en/wiki/Linux_I/O_Performance_Tests_using_dd)
