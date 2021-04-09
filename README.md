# Overview
Documentation on network tools useful for quick performance analysis by ordinary developers. These are the some of the tools a network engineer may ask you to perform first when reporting an issue. These area curated set of common tools for diagnosising network issues between two servers that you suspect there may be a problem at. 

These tools are directional, so the results x->y is typically different than y->x. Ideally these tools are performed on both x->y and y->x for a proper analysis. 

### Throughput 
- How much data can get there and how much gets lost? What is the network speed between two points?

1. [iperf3](#iperf3) - Creates client/server between two network computers and measures throughput
2. [dd Netcat](#dd-Netcat) - Copies files between two network computers and measures file transfer (throughput)

### Connectivity 
- Is there a route from x to y and what are the stops between them? What path is taken and is there an issue inside or outside of the network?

3. [traceroute](#traceroute) - Tracks all computers (typically routers) between two locations and measures round trip of sending three packets to each one
4. [mtr](#mtr) - Sends packets to all computers (typically routers) between two locations and reports packet loss and round trip statistics. Considered a combination of ping and traceroute. 
4. [ping](#ping) - Simple tool that sends packets to a remote host and reports round trip time for packets sent to that host. 
5. [nc](#nc) - Check if ip and ports are open and available for traffic

#### Misc observations with traceroute and mtr

1. These methods are more effective with the public internet than internal networks. For example, internal networks that use MPLS (Multi Protocol Label Switching) do not play well with traceroutes and by design, traceroutes can not see each path while traversing them and will return * * * as results.

example:

 1  router1  4.261 ms  4.184 ms  5.198 ms
 
 2  * * *
 
 3  * * *
 
 4  * * *

2. Sometimes a single device may show packet loss. This is potentially not true packet loss, but 'superficial' packet loss, as the loss does not continue down on each hop, but is isolated to a single device. 
True packet loss will begin at one hop, and not only occur at each hope down to the end destination but will increase at each hop as well. So again, as long as these drops are isolated to the one device and not consistent across all subsequent hops, then what is observed is most likely a symptom of ICMP being deprioritized (superficial) by that one router for more important transit traffic. 

example:
              Loss% Snt Last Avg Best Wrst StDev
              
3.|-- router3 0.0% 10 0.8 0.6 0.5 0.8 0.0

4.|-- router4 80.0% 10 4.7 3.7 2.7 4.7 1.4 <-

5.|-- router5 0.0% 10 1.0 1.1 1.0 1.2 0.0

# Tools

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


## dd Netcat 

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

## traceroute 

3. [traceroute](https://github.com/openbsd/src/blob/master/usr.sbin/traceroute/traceroute.c)
* Tracks steps or hops between the source computer and the ip/url entered. Sends three packets to each computer and measures return time. 
* Language - C
* Authors - Linux/OpenBSD

### Install

```Bash
$ sudo apt install traceroute	#Debian/Ubuntu
$ sudo yum install traceroute	#RHEL/CentOS
$ sudo dnf install traceroute	#Fedora 22+ 
```

### Usage

1. Run a traceroute to the server url or ip in question

```Bash
traceroute 8.8.8.8
```

2. Oberve and track path. The output will resemble the following:

![traceroute](https://github.com/peterlamar/network-performance-tools/blob/master/img/traceroute2.png)

3. The key to track this output:

| Hop #        |Name/IP Address | RTT 1 (round trip time)| RTT 2     | RTT 3    |
| -------------|----------------|------------------------|-----------|----------|
| 1            | 174.111.103.xx | 44.904 ms              | 44.885 ms | 44.855 ms      

* Hop number - hop along the route, in this case the first hop
* Domain/IP - IP address or domain of the router. Can appear as * * * if router will not reveal itself
* RTT columns - Display round trip time (RTT) for the packet to reach router and return to your computer. Listed in three columns and sent three times to guage consistency. Look for wildly different times when looking for problems. 

### Pros
* Likely to be already installed

### Cons 
* Typically difficult to act on knowledge if hops are in a network outside of your control. 

### Reference
* More [detail](https://www.inmotionhosting.com/support/website/how-to/read-traceroute)

## mtr 

4. [mtr](https://github.com/traviscross/mtr)
* Combines traceroute and ping. Sends packets to all computers between two locations and reports lost percentage and statistics on packet round trip times. Typically one must perform a mtr from both computers (A -> B) and (B -> A) to observe where packets are being dropped. 
* Language - C
* Authors - https://github.com/traviscross

### Install

```Bash
$ sudo apt install mtr-tiny	#Debian/Ubuntu
$ sudo yum install mtr	#RHEL/CentOS
$ sudo dnf install mtr	#Fedora 22+ 
```

### Usage

1. Run a mtr to the server url or ip in question. MTR will continue to run unless control-C is pressed. 

```Bash
mtr 8.8.8.8
```

2. Alternatively, a report can be generated

```Bash
mtr --report 8.8.8.8
```

which will resemble the following:

![mtr](https://github.com/peterlamar/network-performance-tools/blob/master/img/mtr.png)

### Pros
* Great statistics on network performance

### Cons 
* Typically have to install on destination servers

### Reference
* More [detail](https://www.linode.com/docs/networking/diagnostics/diagnosing-network-issues-with-mtr/#network-diagnostics-background)

## ping 

5. [ping](https://github.com/iputils/iputils)
* Simple tool that sends packets to a remote host and reports round trip time for packets sent. You will have to hit control+c to interrupt the process when you are done with the results. 
* Language - C
* Authors - Linux

### Install

Ping will typically be already installed in most Linux distros

```Bash
$ sudo apt install iputils-ping	#Debian/Ubuntu
$ sudo yum install iputils	#RHEL/CentOS
$ sudo dnf install iputils	#Fedora 22+ 
```

### Usage

1. Run a ping to the server url or ip in question. Ping will continue to run unless control-C is pressed. 

```Bash
ping 8.8.8.8
```

which will resemble the following:

![ping](https://github.com/peterlamar/network-performance-tools/blob/master/img/ping.png)

### Pros
* Lets you know if a server is reachable and what the round time is for a packet

### Cons 
* Very basic information and more detail is often needed

### Reference
* More [detail](https://www.linode.com/docs/tools-reference/linux-system-administration-basics/#the-ping-command)

## NC

### Usage

```
nc -z -v [hostname/IP address] [port number]
```

Example

```
nc -z -v 192.168.10.12 22
Connection to 192.118.20.95 22 port [tcp/ssh] succeeded!
```

### Reference
* More [detail](https://www.thegeekdiary.com/how-to-test-porttcp-udp-connectivity-from-a-linux-server/)
