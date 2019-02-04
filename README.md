# network-performance-tools
Documentation on network tools useful for performance analysis

## Tools 

1. [iperf3](https://github.com/esnet/iperf)
* Language - C
* Authors - Esnet @ Berkely

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

2. Connect to the server with your laptop or some other client and observe network performance statistics

```Bash
iperf3 -c 
```

![iperf](https://github.com/peterlamar/network-performance-tools/blob/master/img/iperf3.png)

### Pros
Mature tool, been around since 2008
### Cons 
Can be difficult to install on older machines which do not support iperf dependencies in thier kernel Centos < 7

### Reference
Advanced [usage](https://www.tecmint.com/test-network-throughput-in-linux/)
