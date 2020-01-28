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


