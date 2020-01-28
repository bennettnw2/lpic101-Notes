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

##### Interface Configuration Files
* Every network interface has its own config file in the `/etc/sysconfig/network-scripts` directory
* Each interface has a configuration file name of, `ifcfg-<interface-name>-X`, where X is the number of the interface starting with 0 or 1 depending upon the naming convention in use.
  * ie: `/etc/sysconfig/network-scripts/ifcfg-eth0` for the first ethernet interface
* Each interface config file is bound to a specific physical network interface(device) by the mac address of the interface

##### Network Interface Naming Conventions
* there are different ways to name your interfaces like:
  * eth0, eth1
  * eno1
  * enp0s3











.
