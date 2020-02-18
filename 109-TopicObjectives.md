Topic 109: Networking Fundamentals
109.1 Fundamentals of internet protocols
Weight: 4

Description: Candidates should demonstrate a proper understanding of TCP/IP network fundamentals.

Key Knowledge Areas:

Demonstrate an understanding of network masks and CIDR notation.
Knowledge of the differences between private and public "dotted quad" IP addresses.
Knowledge about common TCP and UDP ports and services (20, 21, 22, 23, 25, 53, 80, 110, 123, 139, 143, 161, 162, 389, 443, 465, 514, 636, 993, 995).
Knowledge about the differences and major features of UDP, TCP and ICMP.
Knowledge of the major differences between IPv4 and IPv6.
Knowledge of the basic features of IPv6.
The following is a partial list of the used files, terms and utilities:

/etc/services
IPv4, IPv6
Subnetting
TCP, UDP, ICMP


109.2 Persistent network configuration
Weight: 4

Description: Candidates should be able to manage the persistent network configuration of a Linux host.

Key Knowledge Areas:

Understand basic TCP/IP host configuration.
Configure ethernet and wi-fi network using NetworkManager.
Awareness of systemd-networkd.
The following is a partial list of the used files, terms and utilities:

/etc/hostname
/etc/hosts
/etc/nsswitch.conf
/etc/resolv.conf
nmcli
hostnamectl
ifup
ifdown


109.3 Basic network troubleshooting
Weight: 4

Description: Candidates should be able to troubleshoot networking issues on client hosts.

Key Knowledge Areas:

Manually configure network interfaces, including viewing and changing the configuration of network interfaces using iproute2.
Manually configure routing, including viewing and changing routing tables and setting the default route using iproute2.
Debug problems associated with the network configuration.
Awareness of legacy net-tools commands.
The following is a partial list of the used files, terms and utilities:

ip
hostname
ss
ping
ping6
traceroute
traceroute6
tracepath
tracepath6
netcat
ifconfig
netstat
route


109.4 Configure client side DNS
Weight: 2

Description: Candidates should be able to configure DNS on a client host.

Key Knowledge Areas:

Query remote DNS servers.
Configure local name resolution and use remote DNS servers.
Modify the order in which name resolution is done.
Debug errors related to name resolution.
Awareness of systemd-resolved.
The following is a partial list of the used files, terms and utilities:

/etc/hosts
/etc/resolv.conf
/etc/nsswitch.conf
host
dig
getent

# Netstat and ss

What command, depending on its options, can display the open TCP connections, the routing tables, as  well as network interface statistics? (Specify only the command without any path or parameters.)
Correct Answer: netstat

Netstat shows up in the Networking section and the Security section.  In the security section you would use netstat to view active connections; `netstat -tunalp`
* -t means show tcp connections
* -u means show udp connections
* -n means show the network addresses as numbers (default is to show them symbolically)
* -a means show the state of all sockets; includes processes that are used by server processes
* -l means to show us connections that are listening
* -p means show us process ID and the name of the protocol
  * `PID/Program` is the header that shows up; should read it as `PID/Protocol`
* -r will show the routing tables
  * when used with -a to show protocol-cloned routes
  * if -s is present, it will show routing statistics
  * if -l is present, we will also get the maximum transmission unit (mtu) will also be displayed

So in checking out the man pages along with running commands with different flags, I understand netstat to be a multi-use command.  What I mean by that is that there are a few base commands that will show you different things and then there are the options or other flags that go along with the base command.  For instance if you run `netstat -tunalpr`, you will only get the as routing table indicated by the -r flag.  Or if you run `netstat -tuas` you will only get the statistics given by the -s command.  But as this command has been deprecated for `ss`  you should use `ss` instead.


`ss` (short for socket statistics)is not able to show routing tables.  Many of the switches in `ss` are the same as `netstat`

******************************

# IP Address CIDR Notation

For solving problems regarding how many IP addresses can be used for unique hosts in sub-netting just write out a small table of numbers.
The top row will be binary bit notation?  128 64 32 16 8 4 2 1 and the bottom row will be whole numbers, in ascending order to 32 I guess these are positional numbers which make sense because when you have a /28, for instance, all the bits before it, 1-27 are all fixed and cannot be changed.
```
128 64 32 16 8 4 2 1 | 128 64 32 16 8  4  2  1  | 128 64 32 16 8  4  2  1  | 128 64 32 16 8  4  2  1
1   2  3  4  5 6 7 8 | 9   10 11 12 13 14 15 16 | 17  18 19 20 21 22 23 24 | 25  26 27 28 29 30 31 32
```
Since each octet of an IP address has 8 spots; 32/8 = 4 which are the four octets of an IP address.  Depending on the cidr number will determine up to what position of the bits are fixed.

For the last octet of:
```
128 64 32 16 8  4  2  1
25  26 27 28 29 30 31 32
```

The bit notation corresponds to how many IP addresses we can have. NOTE:  One will always be reserved for the broadcast address and the other is and the other for the network address.  The first address in the range is the network address and the last address in the range is the broadcast address.


******************************

# Determine Subnet Mask Address

For the subnet mask you would take the total number of IP addresses, which if you had a chart would just double starting with the furthest left number take one from it, and then take that resulting number from 255; 

For example a /29 has a total of 8 IP addresses
* 8 - 1 = 7; 255 - 7 = 248
* therefore 255.255.255.248 would be the subnet mask

Q: What would be the broadcast address of a /29?

Q: What do you do if the / is larger than a /25? or 256 addresses?
A: The divisions of / and the IP octets are 1-8, 9-16, 17-24, 25-32; Multiply the total of IP addresses by and respective to the / number; this is getting to be a bit complicated...

On second thought, there is a pattern that if you know the method above for the last octet, just map the answer positionally and make sure depending on the / number, you mark that following octet as 0

For example; a /14 and using this chart below...
```
128 64 32 16 8 4 2 1 | 128 64 32 16 8  4  2  1  | 128 64 32 16 8  4  2  1  | 128 64 32 16 8  4  2  1
1   2  3  4  5 6 7 8 | 9   10 11 12 13 14 15 16 | 17  18 19 20 21 22 23 24 | 25  26 27 28 29 30 31 32
```
I would start with what I know which is 255.x.0.0.  Then I know that the /14 is in the same position as the /30.
A /30 gives me 4 ip addresses and using the formula above for subnet masks, 4-1 = 3; 255-3 = 252. The final subnet mask will be 255.252.0.0

******************************

# /etc/nsswitch.conf file
I am very unsure of what this even does.  Wikipedia says, "...NSS is a facility that provide a variety of sources for common configuration databases and name resolution mechanisms.  These sources include local operating system files, DNS, NIS, and LDAP."

What?!  So this file will direct a service to search for the answer at a variety of different sources?  Like, if you have `passwd: db files` will it use the `/etc/passwd` file as the db or the file?  How does the nsswitch.conf file know where to look if it just says file or db?

Wikipedia continues to say that /etc/nsswitch.conf will list the databases such as passwd, shadow and group, and then one or more sources for obtaining that information.  Those sources can be local files, LDAP (Lightweight Directory Access Protocol), NIS (Network Information Service), NIS+ and `wins` (Windows Internet Name Service).

Maybe I am starting to understand.  I guess what I don't understand is the mechanics of this...  Like my question above.  How does `file` translate into an actual location.  I guess I don't need to know for the test but I am curious like a cat.  So bottom line is that this file tells the system where to find information that it is searching for and since this falls in the Networking Fundamentals section, I guess this is for the `hosts:` entry which the values are `files dns`.

`hosts:     files dns`

In the above example, when the computer needs to resolve a hostname to an ip address, first it will look into files like `/etc/hosts` and `/etc/hostname`.  Next it would look into the DNS entries in `/etc/resolv.conf` if it did not find the answer in the local files.

Therefore, for the exam, I think they are just testing to see if you know what valid entries there can be for a entry.  From the man page:
```
# Valid entries include:
#
#	nisplus			Use NIS+ (NIS version 3)
#	nis			Use NIS (NIS version 2), also called YP
#	dns			Use DNS (Domain Name Service)
#	files			Use the local files in /etc
#	db			Use the pre-processed /var/db files
#	compat			Use /etc files plus *_compat pseudo-databases
#	hesiod			Use Hesiod (DNS) for user lookups
#	sss			Use sssd (System Security Services Daemon)
#	[NOTFOUND=return]	Stop searching if not found so far
```

In addition to knowing what the possible databases can be, like:
```
* hosts       * ethers
* shadow      * netmasks
* passwd      * networks
* netgroup    * protocols
* automount   * rpc
* services    * group
* bootparams  * netgroup
```

******************************

# Route Command
What does the `route` command do?  What even is a route?  I think it is the path/directions you give the computer to look for how to direct traffic?  Like, either internal traffic or external traffic.  Maybe?  Let see, shall we?  In my notes I have that:
* `route` is a utility that will display the routing table.
* Using the `route` utility you can add and delete routes
  * the syntax would look like:
    * `route add -net 192.168.49.0 netmask 255.255.255.0 gw 192.168.1.25`
    * this will add another route to the routing table.

I still don't get it.  I guess I should get a grip on what is a route, for reals.  From [study-ccna.com](https://study-ccna.com/what-is-ip-routing/) says that *IP Routing* is the process of sending packets from a host on one network to another host on a different remote network.  That makes sense.  It continues to say that routing is done by routers (obvi) and that routers examine the destination IP address of a packet and then determines the next hop address to get the data closer to its destination.  Routers use a routing table to determine what the next hop will be to get the packet to the proper location.

It seems to me that the route command adds, removes and just plain edits routing tables so that the computer will know where to send packets?  According to the man page, Yes!

> **`route - show / manipulate the IP routing table`**
> 
> Route manipulates the kernel's IP routing tables.  Its primary use is to set up static  routes  to  specific hosts or networks via an interface after it has been configured with the ifconfig(8) program.
> 
> When the add or del options are used, route modifies the routing tables.  Without these options, route displays the current contents of the routing tables.

For the exam I would need to know how to use the routing table to:
* view and change routing tables
* setting the default route using `iproute2`
* be aware of any legacy route tools

Q: what is the old way of configuring a routing table.  Seems to me that the new way is with `iproute2`
A: It is just plain ol' route. from the man page again
> This program is obsolete. For replacement check ip route.

Well how do I view and change routing tables?
`route` will display a routing table
```
$ route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         gw-li682.linode 0.0.0.0         UG    100    0        0 eth0
23.239.9.0      0.0.0.0         255.255.255.0   U     100    0        0 eth0
```
* Destination is the destination of the network or destination host
* Gateway is the gateway address or '\*' if none is set
* Genmask is the network mask for the destination net; 255.255.255.255 for a host destination and 0.0.0.0 for the default route
* Flags can mean:
  * U (route is up)
  * H (target is a host)
  * G (use gateway)
  * R (reinstate route for dynamic routing)
  * D (dynamically installed by daemon or redirect)
  * M (modified from routing daemon or redirect)
  * A (installed by addrconf)
  * C (cache entry)
  * !  (reject route)
* Iface is the interface to which packets will be sent through for the specific route will be sent

Not sure what all the above means still but it is ok.  Maybe subconsciously. I can build on the subconscious.

How do I change a routing table?  You can delete or add an entry and that would look like:
`route add -net 192.56.76.0 netmask 255.255.255.0 metric 1024 dev eth0`
* the above adds a route to the local network 192.56.76.x via "eth0"
or `route add -net 192.168.49.0 netmask 255.255.255.0 gw 192.168.1.25`
* -net means the target is a network
* -host means the target is a host
* netmask is the netmask to be used when adding a network route
* gw means to route packets via a gateway
  * NOTE: the specified gateway must be reachable first.  This usually mans that you have to set up a static route to the gateway beforehand.  If you specify the address of one of your local interfaces, it will be used to decide about the interface to which the packets should be routed to.
`route del default` will delete the default route which is labeled either "default" or "0.0.0.0" in the destination field of the current routing table

Well that was fun, I still don't really understand what is going on and just have a vague Idea, maybe I'll watch some youTube videos...

First I want to check out setting a default route with iproute2
Basically, it is the same but a bit different syntax.  Instead of `route...`, the command will be `ip route...`. This reminds me of the `linode-cli` utility where you can build on commands.  `ip` is the base command and then you select the object you want to manipulate or create like, `addr`, `link`, `route`, `maddr` or `neigh`.  Then you select the command you want to run and the commands available to you differ depending on the object you select.  I think for the exam I'll only really need to know `addr`, `link` and `route`.

For the differences between `route` and `iproute2`:
* `route` would look like `route add -net 192.168.1.0 netmask 255.255.255.0 dev eth0`
* `iproute2` would look like `ip route add 192.168.1.0/24 dev eth0`
or
`route add default gw 192.168.1.1` vs `ip route add default via 192.168.1.1`

That is it for now.  I am going to give my brain a rest from the route command and work on configuring a network from scratch.  Maybe that will help to put things in perspective.

******************************

# Configuring a network from scratch
Q: How do I even set up a cloud instance with no networking?  That's my first step...

For now I am going to go over the first search result I found on [Opensource.com](https://opensource.com/life/16/6/how-configure-networking-linux)

There are two startup services; `network` startup and `NetworkManager`  You can use your home computer as a router if you wanted to!

### Interface Configuration Files
* Every network interface has its own config file in the `/etc/sysconfig/network-scripts` directory
* Each interface has a configuration file name of, `ifcfg-<interface-name>-X`, where X is the number of the interface starting with 0 or 1 depending upon the naming convention in use.
  * ie: `/etc/sysconfig/network-scripts/ifcfg-eth0` for the first ethernet interface
* Each interface config file is bound to a specific physical network interface(device) by the mac address of the interface

### Network Interface Naming Conventions
* there are different ways to name your interfaces like:
  * eth0, eth1
  * eno1
  * enp0s3

### Config File Examples

This example network interface configuration file, `ifcfg-eth0`, defines a static IP address configuration for a CentOS 6 server installation.
```
# Intel Corporation 82566DC-2 Gigabit Network Connection
DEVICE=eth0
HWADDR=00:16:76:02:BA:DB
ONBOOT=yes
IPADDR=192.168.0.10
BROADCAST=192.168.0.255
NETMASK=255.255.255.0
NETWORK=192.168.0.0
SEARCH="example.com"
BOOTPROTO=static
GATEWAY=192.168.0.254
DNS1=192.168.0.254
DNS2=8.8.8.8
TYPE=Ethernet
USERCTL=no
IPV6INIT=no
```
This file starts the interface on boot (ONBOOT=yes), assigns the interface a static IP address (IPADDR=), defined a domain and a network gateway, specifies two DNS servers, and does not allow not-root users to start and stop the interface.

This next interface config file, `ifcfg-eno1`, provides a DHCP configuration for a desktop workstation.
```
TYPE=Ethernet
BOOTPROTO=dhcp
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=no
IPV6_AUTOCONF=no
IPV6_DEFROUTE=no
IPV6_FAILURE_FATAL=no
NAME=eno1
UUID=a67804ff-177a-4efb-959d-c3f98ba0947e
ONBOOT=yes
HWADDR=E8:40:F2:3D:0E:A8
IPV6_PEERDNS=no
IPV6_PEERROUTES=no
```
In this configuration file, the DHCP entries, IP adrdress, the search domain and all other network information are not defined because they are supplied by the DHCP server.  

Note that in both interface configuration files, the HWADDR line specified the MAC address of the physical network interface.  If you change the network device, you will have to change this line in the configuration file.

### Configuration Options
* DEVICE: The logical name of the device, such as eth0 or enp0s2.
* HWADDR: The MAC address of the NIC that is bound to the file, such as 00:16:76:02:BA:DB
* ONBOOT: Start the network on this device when the host boots. Options are yes/no. This is typically set to "no" and the network does not start until a user logs in to the desktop. If you need the network to start when no one is logged in, set this to "yes".
* IPADDR: The IP Address assigned to this NIC such as 192.168.0.10
* BROADCAST: The broadcast address for this network such as 192.168.0.255
* NETMASK: The netmask for this subnet such as the class C mask 255.255.255.0
* NETWORK: The network ID for this subnet such as the class C ID 192.168.0.0
* SEARCH: The DNS domain name to search when doing lookups on unqualified hostnames such as "example.com"
* BOOTPROTO: The boot protocol for this interface. Options are static, DHCP, bootp, none. The "none" option defaults to static.
* GATEWAY: The network router or default gateway for this subnet, such as 192.168.0.254
* ETHTOOL_OPTS: This option is used to set specific interface configuration items for the network interface, such as speed, duplex state, and autonegotiation state. Because this option has several independent values, the values should be enclosed in a single set of quotes, such as: "autoneg off speed 100 duplex full".
* DNS1: The primary DNS server, such as 192.168.0.254, which is a server on the local network. The DNS servers specified here are added to the /etc/resolv.conf file when using NetworkManager, or when the peerdns directive is set to yes, otherwise the DNS servers must be added to /etc/resolv.conf manually and are ignored here.
* DNS2: The secondary DNS server, for example 8.8.8.8, which is one of the free Google DNS servers. Note that a tertiary DNS server is not supported in the interface configuration files, although a third may be configured in a non-volatile resolv.conf file.
* TYPE: Type of network, usually Ethernet. The only other value I have ever seen here was Token Ring but that is now mostly irrelevant.
* PEERDNS: The yes option indicates that /etc/resolv.conf is to be modified by inserting the DNS server entries specified by DNS1 and DNS2 options in this file. "No" means do not alter the resolv.conf file. "Yes" is the default when DHCP is specified in the BOOTPROTO line.
* USERCTL: Specifies whether non-privileged users may start and stop this interface. Options are yes/no.
* IPV6INIT: Specifies whether IPV6 protocols are applied to this interface. Options are yes/no.

If the DHCP option is specified, most of the other options are ignored. The only required options are BOOTPROTO, ONBOOT and HWADDR. Other options that you might find useful, that are not ignored, are the DNS and PEERDNS options if you want to override the DNS entries supplied by the DHCP server.

******************************

I am launching a Linode and breaking and fixing stuff related to networking.  Here are my thoughts and notes for future me.
* installing `lynx`
  * `dnf config-manager --set-enabled PowerTools`
  * `dnf -y install lynx`

Lynx works just fine and although I could not go to google.com, (too many request error), I went to [nygel.ninja](http://nygel.ninja) instead

I am using the Linux in Action boot and the chapter for network trouble shooting.  It starts with talking about TCP/IP, NAT addressing, Subnets and network masks.  Then it presents a scenario of having to go and trouble shoot a network.

Things to check in order from closest to you to furthest from you.
1. IP Address
2. Router
3. DNS
4. Actual Internet is Down

##### 1. IP Address
How to check the IP Address
* Interface
  * `ip address` | `ip addr` | even `ip a` will display my network interface devices
  ```
  [root@li287-172 ~]# ip addr
  1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
      link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
      inet 127.0.0.1/8 scope host lo
         valid_lft forever preferred_lft forever
      inet6 ::1/128 scope host
         valid_lft forever preferred_lft forever
  2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
      link/ether f2:3c:92:e0:2e:f4 brd ff:ff:ff:ff:ff:ff
      inet 66.228.37.172/24 brd 66.228.37.255 scope global dynamic noprefixroute eth0
         valid_lft 82876sec preferred_lft 82876sec
      inet6 2600:3c03::f03c:92ff:fee0:2ef4/64 scope global dynamic noprefixroute
         valid_lft 2591996sec preferred_lft 604796sec
      inet6 fe80::f03c:92ff:fee0:2ef4/64 scope link noprefixroute
         valid_lft forever preferred_lft forever
  ```
    * the `ip` command is a cool command in that it encompasses many different things you can do with it.  Prior to `ip` there was a suite of different commands you would use for networking but I guess they all got wrapped up into `ip`
    * the syntax for ip is `ip [OPTIONS] OBJECT {COMMAND | help} ` 
    * options are optional []; we need an object and either a command or the string help.
    * typing help after the object will give you the commands for the object
    * and i think you can also type help after a command of an object and it will give you help on that
      * nope.  I will only give you help upto the command level but the help does show a syntacical example of the object and the command

I know I have a working IP address as eth0 shows a state of `UP` and I see an IP address in inet and inet6.  How do I break this so that I can fix this?  How is an ip address configured to "work" in a network.  I bet you use the IP command.  Let me see what I can find with `ip help`  It may be link, address, or route.

I am reading the section Assigning IP addresses
I playing around with the commands I come across this one which I think is important
```
[root@li287-172 ~]# ip route list
default via 66.228.37.1 dev eth0 proto dhcp metric 100
66.228.37.0/24 dev eth0 proto kernel scope link src 66.228.37.172 metric 100
```

I then delete the ip address (I think that would break network connectivity!) and It did!  I was kicked out.
```
[root@li287-172 ~]# ip addr delete 66.228.37.172 dev eth0
packet_write_wait: Connection to 66.228.37.172 port 22: Broken pipe
```

So now I am going into the weblish console and running some commands there to see if I can get the networking to work again.

I attempted to see the route again with `ip r list` but did not get any output!  I tried to go to google.com and nygel.ninja with lynx and I got an error there too.  The network is definitely borked.

What to do now?  Let's reverse our command and add the IP address back.
`[root@li287-172 ~]# ip addr add 66.228.37.172 dev eth0`
  * NOTE: this is how to configure a static IP address
I ran `ip r list` again but I still did not get any output.  Turns out, I needed to restart the NetworkManager for the settings to set
`systemctl restart NetworkManger`
* this is on Centos 8 and 'NetworkManager' is case sensitive

So I accomplished the task of removing and replacing the IP address now I would like to focus on the mechanics of what, why and how this happens.  But for now, I'll just continue on in this chapter

Well it seems in continuing on in the chapter I get a glimpse into the answer above.  If my understanding is correct, I removed the connection (route?) from the device, eth0

What is a route?  It looks like a route is the way the operating system will find the location? of the network to connect to?  I am not sure in my understanding of a route.  But, according to this, if the OS can already see its way through to a working network, then `ip route` will show me the systems routing table.  The routing table will show how? to get to the local network and also the IP address of the device that it will use as a gateway router.
```
[root@li287-172 ~]# ip route
default via 66.228.37.1 dev eth0 proto dhcp metric 100
66.228.37.0/24 dev eth0 proto kernel scope link src 66.228.37.172 metric 100
```
This first line is the address of the gateway router which the local machine can connect to external networks
* `66.228.37.1` is the gateway address
* I think it is saying, "This is the default gateway route and you can connect via 66.228.37.1.  This route is configured to the device eth0 and assigned by dhcp."  I don't know what metric 100 means.

The second line shows the NAT network and the netmask
* `66.228.37.0/24` this is the address of the local NAT network

NOTE: the default IP address assigned to routers is x.x.x.1. For instance, if you have a computer on your local network with an IP address of 192.168.1.34, chances are that router's address will be 192.168.1.1.  In this case, the gateway address is the same as the router address so then the gateway is the router?

I am going to watch a youtube video on routes and the `ip` command to see if my understaning is correct or not.  brb [YouTube Video - IP Routing Explained](https://www.youtube.com/watch?v=8qtKpZGoNdI)

Right off the bat, I now understand that IP packet forwarding and routing are the same thing.  Which makes sense when you think about it!

The first thing a computer will do when it has a packet to forward is to see if the destination is in the local network or external network.  It will first look at it's own IP address and the subnet mask to make that determination.  If it determines to be external, it will then look to the gateway device?

Wow, that is a good video but it is super in-depth for my purposes.  I am just trying to get a broad, working overview that I can use as a basics.

[YouTube Video - Linux - Network Configuration](https://www.youtube.com/watch?v=Yr6qI6v1QCY)
find all the networking stuff; release and renew DHCP; 

I think one will be great as it is Linux centered and the speaker is using virtual box to virtualise their environment which is something I wanted to do so I could have a bit more control of the network.  I am not sure what Linode does to get networking going on a Linode.  Anyhow, it is time for my sleep time.


* Route
* IP config (Static or DHCP)

******************************

##### Mon Feb 10 20:16:37 EST 2020
## 109.4 Configure Client Side DNS

To make a query to a DNS server, (which is how you will be able to connect to the inter and intra nets) your computer needs to know where to send a request to.  The /etc/resolv.conf is where your system will look for the answer.  This file is used to find a DNS server, to ask for an ip address for whatever domain it is that you want.  The service or application requesting the IP address; will then contact the name server at the address listed in /etc/resolv.conf.

`/etc/resolv.conf` is where the computer will look to find where to look for dns entries.  This is what it will use to make external connections to the outside world.

`/etc/hosts` is what the computer will use to make local connections to other computers within the same network.

109.4 Configure Client Side DNS
I am going to create a quick computing "kata" for these Key Knowledge Areas:
* Query remote DNS servers.
* Configure local name resolution and use remote DNS servers.
* Modify the order in which name resolution is done.
* Debug errors related to name resolution.
* Awareness of systemd-resolved.

#### Query remote DNS servers
* dig MX nygel.ninja
* dig @8.8.8.8 nygel.ninja
* host nygel.ninja
* host nygel.ninja 8.8.8.8
* dig -t any nygel.ninja
* host -a nygel.ninja

#### Configure local name resolution and use remote DNS servers
https://debian-handbook.info/browse/stable/sect.hostname-name-service.html
* This uses `/etc/resolv.conf` and `/etc/hosts/` as well as `/etc/nsswitch`

For local name resolution our tools are, `/etc/nsswitch.conf` and `/etc/hosts`.  How they interact is that that your system will look into `/etc/nsswitch.conf` to see where to look up destinations for locations within its own network.

You can looks at `/etc/hosts` as a small table that maps IP addresses and local machine hostnames.  Even if there was no name server on the local network or there is a network outage, this file can be used to navigate between different local machines.

Example of `/etc/hosts`:
```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
2600:3c03::f03c:91ff:fe9f:138d   nodeserver
69.164.214.8   nodeserver
```

The mechanism for local name resolution is the `/etc/nsswitch` file. You will want to make sure you have an entry that looks like this: `hosts   files dns`.  Here is an example of the file:
```bash
#
# /etc/nsswitch.conf
#
# An example Name Service Switch config file. This file should be
# sorted with the most-used services at the beginning.
#
# The entry '[NOTFOUND=return]' means that the search for an
# entry should stop if the search in the previous entry turned
# up nothing. Note that if the search failed due to some other reason
# (like no NIS server responding) then the search continues with the
# next entry.
#
# Valid entries include:
#
#	nisplus			Use NIS+ (NIS version 3)
#	nis			Use NIS (NIS version 2), also called YP
#	dns			Use DNS (Domain Name Service)
#	files			Use the local files
#	db			Use the local database (.db) files
#	compat			Use NIS on compat mode
#	hesiod			Use Hesiod for user lookups
#	[NOTFOUND=return]	Stop searching if not found so far
#

# To use db, put the "db" in front of "files" for entries you want to be
# looked up first in the databases
#
# Example:
#passwd:    db files nisplus nis
#shadow:    db files nisplus nis
#group:     db files nisplus nis

passwd:     files sss
shadow:     files sss
group:      files sss
#initgroups: files sss

#hosts:     db files nisplus nis dns
hosts:      files mdns4_minimal [NOTFOUND=return] dns myhostname

# Example - obey only what nisplus tells us...
#services:   nisplus [NOTFOUND=return] files
#networks:   nisplus [NOTFOUND=return] files
#protocols:  nisplus [NOTFOUND=return] files
#rpc:        nisplus [NOTFOUND=return] files
#ethers:     nisplus [NOTFOUND=return] files
#netmasks:   nisplus [NOTFOUND=return] files

bootparams: nisplus [NOTFOUND=return] files

ethers:     files
netmasks:   files
networks:   files
protocols:  files
rpc:        files
services:   files sss

netgroup:   nisplus sss

publickey:  nisplus

automount:  files nisplus sss
aliases:    files nisplus
```

This line basically says, "To find local hosts, first look into local files. Namely, `/etc/hosts`.  If you don't find a match there then look in the DNS settings under `/etc/resolv.conf`". You can even use NIS/NIS+ or LDAP servers as other possible sources

To configure and use remote DNS Servers you will need to be sure your `/etc/resolv.conf` file is configured correctly.  This is what it should look like and it is normally automatically configured by NetworkManager.
`/etc/resolv.conf`
```bash
# Generated by NetworkManager
search members.linode.com
nameserver 50.116.53.5
nameserver 50.116.58.5
nameserver 50.116.61.5
```

If I am not mistaken you just need to have the word nameserver and then an IP address of a nameserver.  Like `nameserver 8.8.8.8` to use google's nameserver.  Or `nameserver 1.1.1.1` to use Cloudflare's nameserver.  

Q: Can nameserver and DNS server be used interchangeably?
Some answers from Quora:
> The acronym DNS stands for Domain Name System. It is a distributed system for translating host names into IP addresses. The name server is usually what people call the local DNS server,  is typically used to locate a DNS server.  If your website is hosted by another company, sometimes you'll need to use their name servers.
Name servers "point" your domain name to the company that controls its DNS settings. Usually, this will be the company where you registered the domain name.

> Now, I will come to the Name Server.
A name server is the server where DNS software is installed and manage all the domain name records. Name servers are often called DSN (Data Source Name) servers.
Every web site has two name servers to which it is pointed. So, lets say your name servers are ns1.xyz.com and ns2.xyz.com  . So, this is the place where the records for xyz.com would be hosted with the corresponding IP Addresses.

So my understanding is that DNS gets their ip/domainName mapping data from nameservers.

I'd like to take a better look into the mechanics of `/etc/resolv.conf`
https://www.tldp.org/LDP/nag/node84.html
So `search` and potentially `domain` are default domains that are tacked onto a hostname?
And `nameserver` is just an address the computer will use to use a name server to help with DNS queries

#### Modify the order in which name resolution is done
In the `/etc/nsswitch.conf` file, change the order of the parameters of the `hosts` line

#### Debug errors related to name resolution
When solving your DNS issues, always ensure that you first determine whether your DNS server is returning the same response when queried from different locations. You should also ensure that your domain name is active and that you have a stable and robust ISP

Because of all the moving parts and connections involved, a variety of things can go wrong with your DNS:
* Slow updates cause problems that ripple out, often misleading you as you try to fix the issue.
* Incorrect DNS settings create a slow-motion disaster that will not be immediately apparent.
* There are multiple points of failure, and it can be hard to figure out where things have gone wrong.

**Troubleshooting Common Errors**
The first thing you can do when faced with DNS errors is to check for the most common issues:
1. Check your domain registration. Make sure your registration is up to date and paid for, and hasn’t expired. If it is, you’ll have to renew it.
2. Check your nameservers. Make sure that your domain is using the correct nameservers. If you’ve recently switched your domain registrar or hosting company, this is the most likely issue. Your domain will need to point to the correct nameservers for where your website is hosted. You can check your web host’s website to find out which nameservers you should be using.
3. Wait for any recent changes to propagate. Unfortunately, due to the nature of DNS servers, it can take up to 24-48 hours for any changes you make to propagate across the web. If you just corrected your nameservers, give it some time to propagate.

#### Awareness of systemd-resolved
* `systemd-resolved` part of the `systemd` package.
* `systemd-resolved` provides resolver services for DNS.
* The configuration file is located `/etc/systemd/resolved.conf
* I just checked on an instance and it seems to be disabled by default
