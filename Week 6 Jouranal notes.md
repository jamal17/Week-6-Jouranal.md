
# How Linux Works: What Every Superuser Should Know Journal Notes from Chapters 8&9



Chapter 8. A Closer Look at Processes and Resource Utilization

In this chapter we will digging deeper for the relationships between processes, the kernel, and system resources. 

There are three basic kinds of hardware resources: 

1.CPU, memory

2.I/O. Processes 

3.kernel’s job 

Most of the tools we will see in this chapter are helpful.

Tracking Processes

The command of ps is a current process, but this process it not going to be a helpful to determine about the process using too much CPU 
time or memory, for this reason using the top program will be more useful because it will display the status, also it will updating the 
display every second. 


It is very important the top shows the most active processes. 

We use the most important command to send command to the top wi th keystroke.

We have other utilities for Linux, it is similar to top, atop and htop, the  htop has many of abilities of the lsof command describd
in the next section.

Finding Open Files with lsof

lsof command it is open files and the processes using them, lsof is among the most useful tools for finding trouble spots. 

 

Reading the lsof Output

lsof command line usually produces a tremendous amount of output.

$ lsof

COMMAND PID USER  FD TYPEDEVICE      SIZE     NODE NAME

init     1 root cwd  DIR    8,1     4096        2 /

init     1 root rtd  DIR    8,1     4096        2 /

init     1 root mem  REG    8,     47040  9705817 /lib/i386-linux-gnu/libnss_files-2.15.so

init     1 root mem  REG    8,1     42652 9705821 /lib/i386-linux-gnu/libnss_nis-2.15.so

init     1 root mem  REG    8,1     92016 9705833 /lib/i386-linux-gnu/libnsl-2.15.so

--snip--

vi    22728  juser cwd  DIR    8,1   4096 14945078/home/juser/w/c

vi    22728  juser  4u  REG    8,1  1288  1056519 /home/juser/w/c/f

 

Using lsof

There are two basic approaches to running lsof:

1.List everything and pipe the output to a command.

2.Narrow down the list that lsof provides with command-line options.

Note: we can use the command-line provide a filename as an argument and have lsof list only the entries that match the argument

For example, the following command displays entries for open files in /usr:

$ lsof /usr

NOTE: It is very important to updating everything when we update the kernel because the lsof is a dependent on kernel information.

strace

The strace utility prints all the system calls that a process makes. 

To see it in action, we use the below command:

$ strace cat /dev/null

ltrace

The ltrace command tracks shared library calls. 

The output is like that of strace, which is why we’re mentioning it here, but it doesn’t track anything at the kernel level. 

Be warned that there are many more shared library calls than system calls. 

Because there are many shared library calls than system call we need to filter the output.

Threads

In Linux, some processes are divided into pieces called threads, all threads inside a single process share their system resources
and some memory.

 

NOTE: Normally, we won’t interact with individual threads as we would process.

Threads can confuse things when it comes to resource monitoring because individual threads in a multithreaded process can 
consume resources simultaneously. 

For example, top doesn’t show threads by default; we’ll need to press H to turn it on. 

Measuring CPU Time

We use the -p option to monitor one or more specific processes over time

$ top -p pid1 [-p pid2...]

We will use time to find out how much CPU time a command uses during its lifetime.

But we have to know the most shells have a built-in time command that doesn’t provide extensive statistics, for this reason we 
need to run the command /usr/bin/time. 

For example, to measure the CPU time used by ls, run

$ /usr/bin/time ls

After ls terminates, time should print output like that below. 

The key fields are in boldface:

0.05user 0.09system 0:00.44elapsed 31%CPU(0avgtext+0avgdata 0maxresident)k

0inputs+0outputs(125major+51minor)pagefaults 0swaps


Adjusting Process Priorities

we can change the way the kernel schedules a process in order to give the process more or less CPU time than other processes. 

The kernel runs each process according to its scheduling priority, which is a number between –20 and 20, with –20 being the fore
most priority.

The ps -l command lists the current priority of a process, but it’s a little easier to see the priorities in action with the top command
, as shown here:

$ top

Tasks: 244 total,  2 running,242 sleeping,   0 stopped,   0 zombie

Cpu(s): 31.7%us, 2.8%sy, 0.0%ni, 65.4%id,  0.2%wa,  0.0%hi,  0.0%si,   0.0%st

Mem:   6137216k total,5583560k used,   553656k free,    72008k buffers

Swap:  4135932k total, 694192k used, 3441740k  free,   767640k cached

  PID USER     PR NI  VIRT  RES  SHR S %CPU%MEM    TIME+ COMMAND

28883 bri      20  0 1280m 763m  32m S  58 12.7 213:00.65 chromium-browse

 1175 root     20  0  210m  43m  28m R   44  0.7  14292:35 Xorg

 4022 bri      20  0  413m 201m  28m S   29  3.4   3640:13 chromium-browse

 4029 bri      20  0  378m 206m  19m S    2  3.5  32:50.86 chromium-browse

 3971 bri      20  0  881m 359m  32m S    2  6.0 563:06.88 chromium-browse

 5378 bri      20  0  152m  10m 7064 S    1  0.2  24:30.21 compiz

 3821 bri      20  0  312m  37m  14m S    0  0.6  29:25.57 soffice.bin

 4117 bri      20  0  321m 105m  18m S    0  1.8  34:55.01 chromium-browse

 4138 bri      20  0  331m  99m  21m S    0  1.7 121:44.19 chromium-browse

 4274 bri      20  0  232m  60m  13m S    0  1.0  37:33.78 chromium-browse

 4267 bri      20  0  1102m 844m 11m S    0 14.1  29:59.27 chromium-browse

 2327 bri      20  0  301m   43m 16m S    0  0.7 109:55.65 unity-2d-shell

Using uptime

The uptime command tells us three load averages in addition to how long the kernel has been running:

$ uptime

... up 91 days, ... load average: 0.08, 0.03, 0.01

The three bolded numbers are the load averages for the past 1 minute, 5 minutes, and 15 minutes, respectively. 

Memory

The free command is the most simple way to check the system memory.

How Memory Works

A user process does not actually need all its pages to be immediately available in order to run. 

To see how this works, consider how a program starts and runs as a new process:

1.    The kernel
loads the beginning of the program’s instruction code into memory pages.

2.    The kernel may
allocate some working-memory pages to the new process.

3.    As the process
runs, it might reach a point where the next instruction in its code isn’t in
any of the pages that the kernel

 initially loaded. At this point, the kernel
takes over, loads the necessary pages into memory, and then lets the program

 resume execution.

4.    Similarly, if
the program requires more working memory than was initially allocated, the
kernel handles it by finding 

free pages (or by making room) and assigning them to the process.

Monitoring CPU and Memory Performance with vmstat

The vmstat command is one of the oldest, with minimal overhead. 

$ vmstat 2

procs -----------memory-------------swap-- -----io---- -system-- ----cpu----

r  b  swpd   free   buff  cache  si so   bi   bo  in  cs us sy id wa

 2  0 320416 3027696198636 1072568   0   0   1    1   2   0 15  2 83  0

 2  0 320416 3027288198636 1072564   0   0    0 1182 407636  1  0 99  0

 1  0 320416 3026792198640 1072572   0   0    0   58281 537  1  0 99  0

 0  0 320416 3024932198648 1074924   0   0    0  308 318541  0  0 99  1

 0  0 320416 3024932198648 1074968   0   0  0    0 208 416  0  0 99  0

 0  0 320416 3026800198648 1072616   0   0   0    0 207 389  0  0 100 0

The output falls into categories: procs for processes, memory for memory usage, swap for the pages pulled in and out of swap, 
io for disk usage, system for the number of times the kernel switches into kernel code, and cpu for the time used by different parts 
of the system.

Using iostat

We use the iostat command to shows the statistics for the machine’s current uptime:

$ iostat

avg-cpu:  %user %nice%system %iowait  %steal   %idle

          
4.46   0.01    0.67   0.31    0.00   94.55

Device:        tp s   kB_read/s   kB_wrtn/s  kB_read   kB_wrtn

sda           4.6 7       7.2 8       49.86   9493727 65011716

sde           0.0 0       0.0 0        0.00     1230         0

The avg-cpu  at the top reports the same CPU utilization information .

There are many partitions on the system we use the -p ALL option. 

$ iostat -p ALL

--snip

--Device:              tps       kB_read/s    kB_wrtn/s    kB_read      kB_wrtn

--snip-

sda                   4.67           7.27          49.83   9496139     65051472

sda1                  4.38           7.16          49.51   9352969     64635440

sda2                  0.00           0.00           0.00         6            0

sda5                  0.01           0.11          0.32     141884       416032

scd0                  0.00           0.00           0.00         0            0

sde                   0.00           0.00           0.00      1230            0

sda1, sda2, and sda5 are all partition of the disk sda.

Per-Process Monitoring with pidstat

The command of The pidstat utility allows to see the resource consumption of a process over time in the style of vmstat. 

Here’s a simple example for monitoring process 1329, updating every second:

$ pidstat -p 1329 1

Linux 3.2.0-44-generic-pae(duplex)     07/01/2015     _i686_ (4 CPU) 

09:26:55PM       PID    %usr %system%guest %CPU CPU Command

09:27:03PM      1329    8.00 0.00     0.00 8.00   1 myprocess

09:27:04PM      1329    0.00 0.00     0.00 0.00   3 myprocess

09:27:05PM      1329    3.00 0.00     0.00 3.00   1 myprocess

09:27:06PM      1329    8.00 0.00     0.00 8.00   3 myprocess

09:27:07PM      1329    2.00 0.00     0.00 2.00   3 myprocess

09:27:08PM      1329    6.00 0.00     0.00 6.00   2 myprocess


Chapter 9. Understanding your Network and its Configuration

Network Basics

Any machine connected to the network we called a host., and the host is connected to the router.

The host that can move the data between networks.

The connections on the LAN can be wired or wireless.

Packets

It is the small chunk where the computer transmits data over a network.

The packets have two parts: 

1.a header 

2.payload. 

The header contains identifying information. 

The payload, on the other hand, is the actual application data that the computer wants to send.

(for example, HTML or image data).

Packets allow a host to communicate with others “simultaneously,” because hosts can send, receive, and process packets in any 
order, regardless of where they came from or where they’re going. Breaking messages into smaller units also makes it easier to 
detect and compensate for errors in transmission.

Viewing Your Computer’s IP Addresses

One host can have many IP addresses. To see the addresses that are active on your Linux machine, run

$ ifconfig

There will probably be a lot of output, but it should include something like this:

eth0     Link encap:Ethernet  HWaddr 10:78:d2:eb:76:97

inet addr:10.23.2.4  Bcast:10.23.2.255  Mask:255.255.255.0

inet6 addr: fe80::1278:d2ff:feeb:7697/64 Scope:Link

UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1

RX packets:85076006 errors:0 dropped:0 overruns:0 frame:0

TX packets:68347795 errors:0 dropped:0 overruns:0 carrier:0

collisions:0 txqueuelen:1000
         
RX bytes:86427623613 (86.4 GB)  TX bytes:23437688605 (23.4 GB)

Interrupt:20 Memory:fe500000-fe520000

The ifconfig command’s output includes many details from both the Internet layer and the physical layer. 



Subnets

A subnet is a
connected group of hosts with IP addresses in some sort of order

For example, here are the binary
forms of 10.23.2.0 and 255.255.255.0:

10.23.2.0:       
00001010 00010111 00000010 00000000

255.255.255.0:   
11111111 11111111 11111111 00000000

Now,
let’s use boldface to mark the bit locations in 10.23.2.0 that are 1s in
255.255.255.0:

10.23.2.0:        00001010
00010111 00000010 00000000

 

Common Subnet Masks and CIDR
Notation

There is some easy subnet we my
deal with like 255.255.255.0 or 255.255.0.0, To understand what this means,
look at the mask 

in binary form (as in the example
you saw in the preceding section), all subnet masks are just a bunch of 1s
followed by a bunch 

of 0s. 

For example, you just saw that
255.255.255.0 in binary form is 24 1-bits followed by 8 0-bits. 

 

Routes and the Kernel Routing Table

Connecting Internet subnets is
mostly a process of identifying the hosts connected to more than one subnet.

How does the Linux kernel
distinguish between these two different kinds of destinations? It uses a
destination configuration 

called a routing table to
determine its routing behavior. 

To show the routing table, use
the route -n command. Here’s what you might see for a simple host
such as 10.23.2.4:

$ route
-n

Kernel
IP routing table

Destination    
Gateway     Genmask      
Flags Metric Ref     Use Iface

0.0.0.0        
10.23.2.1   0.0.0.0      
UG    0     
0         0 eth0

10.23.2.0      
0.0.0.0     255.255.255.0 U    
1     
0         0 eth0

 

The Default Gateway

An entry for 0.0.0.0/0 in the
routing table has special significance because it matches any address on the
Internet. 

This is the default route,
and the address configured under the Gateway column (in
the route -n output) in the default route is 

the default gateways. 

.

NOTE

On most networks with a netmask of
255.255.255.0, the router is usually at address 1 of the subnet (for example,
10.23.2.1 in 10.23.2.0/24). Because this is simply a convention, there can be
exceptions.

 

Basic ICMP and DNS Tools

These tools use two protocols of
particular interest: Internet Control Message Protocol (ICMP), which can help  root out 

problems with connectivity and
routing, and the Domain Name Service (DNS) system.

 

ping

ping is the network debugging
tools. 

It sends ICMP echo request packets
to a host that ask a recipient host to return the packet to the sender. 

If the recipient host gets the
packet and is configured to reply, it sends an ICMP echo response packet in 

Return this the way how ping work.

For
example, say that you run ping 10.23.2.1 and get this output:

$ ping
10.23.2.1

PING
10.23.2.1 (10.23.2.1) 56(84) bytes of data.

64
bytes from 10.23.2.1: icmp_req=1 ttl=64 time=1.76 ms

64
bytes from 10.23.2.1: icmp_req=2 ttl=64 time=2.35 ms

64
bytes from 10.23.2.1: icmp_req=4 ttl=64 time=1.69 ms

64
bytes from 10.23.2.1: icmp_req=5 ttl=64 time=1.61 ms

Note: For security reasons, not all hosts
on the Internet respond to ICMP echo request packets, so you might find that
you can connect to a website on a host but not get a ping response.

 

traceroute

to use traceroute host to
see the path our packets take to a remote host. (traceroute -n host will
disable hostname lookups.)

One of the best things
about traceroute is that it reports return trip times at each step in
the route, as demonstrated in this output 

fragment:

4 
206.220.243.106  1.163 ms  0.997 ms  1.182 ms

 5 
4.24.203.65  1.312 ms  1.12 ms  1.463 ms

 6 
64.159.1.225  1.421 ms  1.37 ms  1.347 ms

 7 
64.159.1.38  55.642 ms  55.625 ms  55.663 ms

 8 
209.247.10.230  55.89 ms  55.617 ms  55.964 ms

 9 
209.244.14.226  55.851 ms 55.726 ms   55.832 ms

10 
209.246.29.174  56.419 ms 56.44 ms   56.423 ms

 

DNS and host

IP addresses are difficult to
remember and subject to change, which is why we normally use names such
as www.example.com 

instead. 

The DNS library on the system
normally handles translation automatically between the website name and the IP
address, but 

sometimes we must translate
manually.

 To find the IP address behind a domain name,
use the host command:

$ host
www.example.com

www.example.com
has address 93.184.216.119

www.example.com
has IPv6 address 2606:2800:220:6d:26bf:1447:1097:aa7

 

Manually Adding and Deleting Routes

To
remove a default gateway, run

# route
del -net default

You can easily override the default
gateway with other routes. For example, say your machine is on subnet
10.23.2.0/24, you 

want to reach a subnet at
192.168.45.0/24, and you know that 10.23.2.44 can act as a router for that
subnet. Run this command to 

send traffic bound for 192.168.45.0
to that router:

# route
add -net 192.168.45.0/24 gw 10.23.2.44

You
don’t need to specify the router to delete a route:

# route
del -net 192.168.45.0/24

 

Problems with Manual and
Boot-Activated Network Configuration

Rather than storing the IP address
and other network information on the machine, the machine gets this information
from 

somewhere on the local physical
network when it first attaches to that network. 

 

Network Configuration Managers

There are several ways to automatically
configure networks in Linux-based systems. 

The most widely used option on
desktops and notebooks is NetworkManager. Other network configuration
management systems 

are mainly targeted for smaller
embedded systems, such as OpenWRT’snetifd, Android’s ConnectivityManager
service, 

ConnMan, and Wicd. We’ll briefly
discuss NetworkManager because it’s the one you’re most likely to encounter. 

We won’t go into a tremendous
amount of detail, though, because after you see the big picture, NetworkManager
and other 

configuration systems will be more
transparent.

 

NetworkManager Operation

NetworkManager is a daemon that the
system starts upon boot. Like all daemons, it 

does not depend on a running
desktop component. 

Its job is to listen to events from
the system and users and to change the network 

configuration based on a bunch of
rules.

 

Interacting with NetworkManager

To control NetworkManager from the
command line, we use the nmcli command. 

This is a somewhat extensive
command. 

 

NetworkManager Configuration

configuration directory for
NetworkManager is usually /etc/NetworkManager, and there are
several different kinds of configuration. 

The general configuration file
is NetworkManager.conf. 

 

Resolving Hostnames

One of the final basic tasks in any
network configuration is hostname resolution with DNS. 

DNS differs from the network
elements we’ve looked at so far because it’s in the application layer, entirely
in user space. 

Nearly all network applications on
a Linux system perform DNS lookups. 

The resolution process typically
unfolds like this:

1.   
The application calls a function to look up the IP address behind a hostname.
This function is in the system’s shared library, so the application doesn’t
need to know the details of how it works or whether the implementation will
change.

2.   
When the function in the shared library runs, it acts according to a set of
rules (found in /etc/nsswitch.conf) to determine a plan of action
on lookups. For example, the rules usually say that even before going to DNS,
check for a manual override in the /etc/hosts file.

3.   
When the function decides to use DNS for the name lookup, it consults an
additional configuration file to find a DNS name server. The name server is
given as an IP address.

4.   
The function sends a DNS lookup request (over the network) to the name server.

5.   
The name server replies with the IP address for the hostname, and the function
returns this IP address to the application.

This
is the simplified version. In a typical modern system, there are more actors
attempting to speed up the transaction and/or add flexibility. Let’s ignore
that for now and take a closer look at the basic pieces.

9.12.1
/etc/hosts

On
most systems, you can override hostname lookups with the /etc/hosts file.
It usually looks like this:

127.0.0.1      
localhost

10.23.2.3      
atlantic.aem7.net       atlantic

10.23.2.4      
pacific.aem7.net        pacific

You’ll
nearly always see the entry for localhost here (see 9.13 Localhost).

 

The Transport Layer: TCP, UDP, and
Services

Transport layer protocols bridge the gap
between the raw packets of the Internet layer and the refined needs of
applications. 

The two most popular transport
protocols are the Transmission Control Protocol (TCP) and the User Datagram
Protocol (UDP). 

 

TCP Ports and Connections

TCP provides for multiple network
applications on one machine by means of network ports. 

A port is just a number. 

If an IP address is like the postal
address of an apartment building, a port is like a mailbox number—it’s a
further subdivision.

When using TCP, an application
opens a connection (not to be confused with NetworkManager
connections) between one port on 

its own machine and a port on a
remote host. 

For example, an application such as
a web browser could open a connection between port 36406 on its own machine and
port 80 

on a remote host. From the
application’s point of view, port 36406 is the local port and port 80 is the
remote port.

$ netstat
-nt

Active
Internet connections (w/o servers)

Proto
Recv-Q Send-Q Local Address        
Foreign Address         State

tcp       
0      0
10.23.2.4:47626      
10.194.79.125:5222      ESTABLISHED

tcp       
0      0
10.23.2.4:41475      
172.19.52.144:6667      ESTABLISHED

tcp       
0      0
10.23.2.4:57132      
192.168.231.135:22      ESTABLISHED

 

Establishing TCP Connections

To establish a transport layer
connection, a process on one host initiates the connection from one of its
local ports to a port on a 

second host with a special series
of packets. 

 

Characteristics of TCP

TCP is popular as a transport layer
protocol because it requires relatively little from the application side. 

An application process only needs
to know how to open (or listen for), read from, write to, and close a
connection. 

UDP

UDP is a far simpler transport layer
than TCP, It defines a transport only for single messages; there is no data
stream. 

 

Understanding DHCP

When you set a network host to get
its configuration automatically from the network, you’re telling it to use the
Dynamic Host 

Configuration Protocol (DHCP) to
get an IP address, subnet mask, default gateway, and DNS servers. 

For a host to get its configuration
with DHCP, it must be able to send messages to a DHCP server on its connected
network. 

 

The Linux DHCP Client

Although there are many different
kinds of network manager systems, nearly all use the Internet Software
Consortium (ISC) 

dhclient program to do the
actual work. 

we can test dhclient by
hand on the command line, we must remove any default gateway route.

 

Linux DHCP Servers

we can task a Linux machine with
running a DHCP server, which provides a good amount of control over the
addresses that it 

gives out. 

 

Configuring Linux as a Router

Routers are essentially just
computers with more than one physical network interface. 

we can easily configure a Linux
machine as a router.

For example, say you have two LAN
subnets, 10.23.2.0/24 and 192.168.45.0/24. 

To connect them, you have a Linux
router machine with three network interfaces: two for the LAN subnets and one
for an 

Internet uplink. 

The router’s IP addresses for the
LAN subnets are 10.23.2.1 and 192.168.45.1. When those addresses are
configured, the routing 

table looks something like this
(the interface names might vary in practice; ignore the Internet uplink for
now):

Destination     
Gateway       
Genmask        Flags Metric
Ref     Use Iface

10.23.2.0       
0.0.0.0        255.255.255.0 
U     0     
0       0 eth0

192.168.45.0    
0.0.0.0        255.255.255.0 
U     0     
0       0 eth1

 

Firewalls

Routers should always include some
kind of firewall to keep undesirable traffic out of your network. 

A
system can filter packets when it

1.receives
a packet,

2.sends
a packet, or

3.forwards
(routes) a packet to another host or gateway.

With no firewalling in place, a
system just processes packets and sends them on their way. Firewalls put
checkpoints for packets 

at the points of data transfer
identified above. The checkpoints drop, reject, or accept packets, usually
based on some of these 

criteria:

1.The
source or destination IP address or subnet

2.The
source or destination port (in the transport layer information)

3.The
firewall’s network interface

Firewalls
provide an opportunity to work with the subsystem of the Linux kernel that
processes IP packets. Let’s look at 

that
now.

 

Linux Firewall Basics

In Linux, we create firewall rules
in a series known as a chain. 

A set of chains makes up a table.


As a packet moves through the
various parts of the Linux networking subsystem, the kernel applies the rules
in certain chains to 

the packets. 

For example, after receiving a new
packet from the physical layer, the kernel activates rules in chains
corresponding to input.

These data structures are
maintained by the kernel. The whole system is called iptables, with
an iptables user-space command to 

create and manipulate the rules.

Setting Firewall Rules

Let’s look at how the IP tables
system works in practice. 

Start by viewing the current
configuration with this command:

# iptables
-L

The
output is usually an empty set of chains, as follows:

Chain
INPUT (policy ACCEPT)

target    
prot opt source        destination

Chain
FORWARD (policy ACCEPT)

target    
prot opt source        destination

Chain
OUTPUT (policy ACCEPT)

target    
prot opt source        destination

 

 

Ethernet, IP, and ARP

There is one interesting basic
detail in the implementation of IP over Ethernet that we have yet to cover. 

Recall that a host must place an IP
packet inside an Ethernet frame to transmit the packet across the physical
layer to another host

 

Wireless Ethernet

In principle, wireless Ethernet
(“WiFi”) networks aren’t much different from wired networks. 

Much like any wired hardware, they
have MAC addresses and use Ethernet frames to transmit and receive data, and as
a result 

the Linux kernel can talk to a
wireless network interface much as it would a wired network interface. 

Everything at the network layer and
above is the same; the main differences are additional components in the
physical layer such 

as frequencies, network IDs,
security, and so on.

Unlike wired network hardware,
which is very good at automatically adjusting to nuances in the physical setup
without much 

fuss, wireless network
configuration is much more open-ended. To get a wireless interface working
properly, Linux needs 

additional configuration tools.

Additional components of wireless
networks.

1.Transmission
details. These are physical characteristics, such as the radio
frequency.

we2.Network
identification. Because more than one wireless network can share the
same basic medium, we must be able 

to
distinguish between them. 

3.Management. Although
it’s possible to configure wireless networking to have hosts talk directly to
each other, most 

wireless
networks are managed by one or more access points that all
traffic goes through. 

4.Authentication. You
may want to restrict access to a wireless network. To do so, you can configure
access points to 

require
a password or other authentication key before they’ll even talk to a client.

5.Encryption. In
addition to restricting the initial access to a wireless network, you normally
want to encrypt all traffic 

that
goes out across radio waves.

 

iw

You can view and change kernel
space device and network configuration with a utility called iw. 

To use iw, you normally need
to know the network interface name for the device, such as wlan0. 

Here’s an example that dumps a scan
of available wireless networks. 

# iw
dev wlan0 scan

If the network interface has joined
a wireless network, you can view the network details like this:

# iw
dev wlan0 link

The
MAC address in the output of this command is from the access point that you’re
currently talking to.

.

Wireless Security

For most wireless security setups,
Linux relies on a daemon called wpa_supplicant to manage both
authentication and encryption 

for a wireless network interface. 

Running the daemon by hand every
time we want to establish a connection is a lot of work. In fact, just creating
the configuration 

file is tedious due to the number
of possible options. 

To make matters worse, all of the
work of running iw and wpa_supplicant simply allows your
system to join a wireless physical 

network; it doesn’t even set up the
network layer. And that’s where automatic network configuration managers such
as 

NetworkManager take a lot of pain
out of the process. Although they don’t do any of the work on their own, they
know the 

correct sequence and required
configuration for each step toward getting a wireless network operational.

 

 

References



Ward, Brian. How Linux works: what every superuser should know. 2ND ed. San Francisco: No Starch Press, 2015.171.229
