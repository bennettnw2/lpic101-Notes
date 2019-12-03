# Topic 109 - Networking Fundamentals
## 109.1 Fundamentals of Internet Protocols
  * ### Networking Fundamentals

    * ##### Internet Protocol
      * this is the numerical address that is assigned to a machine
      * these address are like locations
      * these locations are used so that servers can locate, and talk to each other
      * there are two types:
        * IPv4
          * 64 bit with four octets `198.168.10.100`
        * IPv6
          * 128 bit with hexadecimal numbering for addressing

    * ##### Transmission Control Protocol
      * TCP is the method with which all transactions between IP address are communicated
      * this protocol provides a mechanism that transmits and verifies data
      * it confirms that traffic arrives and is assembled correctly on the receiving end
        * relies on send and acknowledgement system (Syn, Ack, SynAck?)
        * each data packet contains a number indicating how the data should be reassembled

    * ##### User Datagram Protocol
      * the UDP protocol is a "stateless" connection between two hosts
      * stateless means there is no protocol to determine the state of the flow of data
      * the data is just sent and assumed to be received correctly
      * because it just sends data rather send data and verify,
        there is less network overhead than a TCP connection
      * good for online gaming, DNS queries and non-critical data

    * ##### Internet Control Message Protocol
      * This protocol is intended for networking equipment to send error messages between themselves
      * Equipment like: routers, switches, firewall and other networking devices
        * often used to query a network device for availability
        * Commands that use ICMP are ping, traceroute
        * it is great to use if something is inoperable

    * ##### IP Class Ranges
      * IP addresses are grouped into Class Ranges
      * RFC 1918 - Describes five ranges that determines how may hosts are available with each class
      ```
      Class   Range       Number of Hosts
      A       1 - 126     16,777,214
      B       128 - 191   65,534
      C       192 - 223   254
      D       224 - 239   Reserved for multicast
      E       240 - 254   Reserved for future research
      ```
      
    * ##### Network Mask
      * this defines a logical network (called a subnet)
      * the subnet indicates the start and end of a range of IP addresses
      * subnetting allows you to break up a class of IP addresses into subclasses aka subnet 
      * Ranges
        * each address class range has an associated network and subnet mask
          * what the heck does this mean?
        * the number following an IP address is known as the bit number
          * it is also known as the Classless Inter Domain Notation (CIDR)
            *eg:
            ```
            Class A: 255.0.0.0/8
              * we use the number 255 in the first octet
              * the zero in the following octets means any number can go there
              * in a class a network, the first number belongs to the network
              * this is known as a "slash 8"
            Class B: 255.255.0.0/16
              * the first two octets define the network
              * the last two octets can be any number
              * it is known as a "slash 16"
            Class C: 255.255.255.0/24
              * has the first three octets defined
              * the last octet is for each individual host on the network
              * this is known as a "slash 24"
              
      * ##### Private IP Address
        * addresses that are used for internal networks
        * prevents the need for every host to have an IP address assigned from a central authority
        * these cannot be routed to the public internet
        * these addresses are only routed within a private network
        ```
        IP Address Range            Number of Hosts    CIDR Notation    Classful Description
        10.0.0.0-10.255.255.255     16,777,216         10.0.0.0/8       single class A network
        172.16.0.0-172.31.255.255   1,048,576          172.16.0.0/12    16 contig class B networks
        192.168.0.0-192.168.255.255  65,536            192.168.0.0/16   256 contig class c networks
        ```

      * ##### Network Gateway and Broadcast Address
        * network gateway is the destination where traffic goes that has no other matching route
        * or traffic that is not intended for the local network
        * you can imagine a network gateway as a gateway to the public internet
          * it is like going out the front door to your house
          * it is like if you need to get something outside your local network, you are able to 
        * broadcast address is the IP address used to broadcast messages to all hosts on a particular network
          * you can think of it like a beacon to ships in the night
            * eg: if 192.168.0.255 is broadcasting, only hosts in the network 192.168.0.0/24 will be able to see the broadcast

      * ##### Common Networking Services
        Learn these by heart!  Create Anki droid list

## 109.2 Persistent Network Configuration
  * ### Using Network Manager to create network connections
    * network manager is used to create and modify various types of network connections
    * There are some advanced stuff you can do with it but we are just focusing on the basics
    * Just need to learn how to connect a standard system to a standard network
    * When working with network manager, be sure to keep these two terms straight:
      * device
        * this is the actual network hardware used to connect a computer to a network
        * network hardware
      * connection
        * this is the config settings of the network device to connect a computer to a network
        * network software

    * ##### `nmcli`
      * "Network Manager Command Line Interface"
      * this is the cli utility used to configure network devices and their connection settings
      * `nmcli dev`
        * `dev` is short for device like a NIC
        * `nmcli dev show` or `nmcli device show` 
          * this output shows the the individual network devices on the system with their settings
          * Important information to look at:
            * the device name at .DEVICE
            * the type at .TYPE
            * the HW address at .HWADDR
              * this is the mac address which should be unique across the world
              * it is an address that uniquely identifies the network card
            * the IP address
              * eg: 10.0.2.15/24 
              * this means the subnet mask is 255.255.255.0 because we have the /24
            * the ROUTE(S)
              * `dst = 0.0.0.0/0` is a catch all IP address for a default route
                * kinda like and external router that will then go to the gateway
              * `dst = 10.0.2.0/24` means that anything that is bound for the local system
                * this keeps the traffic routing local
                * because the device IP address is on the same network as this route ip
            * the Gateway address 
              * remember, this is the address traffic must go through to get to the public internet
            * the DNS entry
              * this is like a telephone book;
              * the device will go to this address to get other addresses locations
          * Network device naming
          ```
          en = ethernet
          wl = wireless
          eno*1* = *onboard* devices index is provided by bios
          ens1 = this is for onboard devices like PCI express hotplug slots
          p = bus; and s = slot
          eth0 = ethernet (older style)
          ```
        * All network devices will have a loopback address
          * IPv4 is 127.0.0.1/0
          * IPv6 is ::1/128
          * this is used in the event it needs to make a network request of itself
          * this is a self, localized interface

        * ##### `nmcli con show`
          * this will show the connections of a system
            * it will give the Name, UUID, and Type of connection
          * remember that we always need to assign our IP addresses and DNS settings to a connection
          * ##### `nmcli con down "Wired connection 1"
            * this will shut down the connection named, "Wired connection 1"
            * use quotes if there are spaces in the connection name
            * this is useful for troubleshooting a security issue and you want to remove the server from the internet
            * if you run `nmcli con show` again, this connections will no longer be highlighted
          * ##### `nmcli con up "Wired connection 1"
            * this will start up the connection again
          * ##### `nmcli con delete "Wired connection 1"
            * this will delete a connection
            * you can then check out the individual device with:
              * `nmcli dev show ens11`
              * this will output the device information
              * you will see that there are no connections associated with it
          * `nmcli con add con-name "backup" type ethernet`
            * this is how you add a connection
              * con-name means we are naming the connection "backup"
              * type means we are creating a ethernet type connection
            * next we will want to add an IP address and there are two ways
              * dhcp - dynamic host configuration protocol
                * if your system is configured to use DHCP, it will send a request to a dhcp service
                * the dhcp service is usually located on a router and ask it for a IP address
                * the dhcp will assign an available IP address to the interface and then you are connected to the network!
              * statically assigning an IP address
                * this is an IP address that is already exists on the interface; it does not change
                * this is useful if the IP address needs to remain the same at all time
              * typically a workstation(client) will use DHCP while an application(server) will use static
              * static assignment of IP address to a connection and interface:
                * `ip4 192.168.122.75/24 gw4 192.168.122.1 ifname ens11 autoconnect`
                  * we give it the ip address with `ip4`
                  * we give it the gateway with `gw4`
                  * we assign it to the interface(device?) with `ifname`
                  * and we can configure to autoconnect on boot with `autoconnect`
              * you can also use `nmcli con edit` for a connection wizard
          * `nmcli con show backup`
            * in taking a look at the new connection we just built, we see we don't have DNS configed
            * we will not be able to surf the web as the DNS request has no where to go
            * we will need to add that in
              * ##### `nmcli con mod backup`
                * we use the mod sub-command and call the name of the connection
                * eg:
                  * `nmcli con mod backup ipv4.dns "192.168.122.1"` 
                  * `nmcli con mod enp0s3 autoconnect y`
                * can verify with this command:
                  * `nmcli -f ipv4.dns con show backup`
                  * => ipv4.dns:       192.168.122.1
      
      * More Network Configuration Commands 
        * ##### `ip`
          * this command can modify IP address and route settings and show interface stats
          * this is from the iproute2 project
          * ##### `ip addr show`
            * this will show us our current interfaces and their respective IP addresses
            * we can see the interface name, state, HW/MAC address, ip/subnet and broadcast
          * ##### `ip route show`
            * this will show us our routing table
            * this can show us if our interface is configured with a dynamic or static IP
          * what if i needed to change an IP address for testing?  Meaning I did not want to make it permanent? 
            * `ip addr add 192.168.122.76/24 dev ens11`
            * then if you take a look at the interface with `ip addr show ens11`
            * we will see both IP addresses
            * you can then delete the old with `ip addr del 192.168.122.75/24 dev ens11`
            * then we are left with the new one
            * next take down the interface with:
              * `ip link set ens11 down`
            * and bring it back up with:
              * `ip link set ens11 up`
            * You should see that the original IP address is now configured

        * ##### `hostnamectl`
          * if you just use `hostname` that will tell you the hostname on a system
          * `hostnamectl set-hostname "ny-host"`
            * this will set your hostname to whatever is inside of the quotes.

  * ### Legacy Networking Tools
    * These are part of the `net-tools` package so be sure to install that
    * ##### `ifconfig`
      * this was the standard networking utility for Linux
      * it has been depreciated in favor of the current `ip` command 
      * this is mostly the same as `ip a`
      * you can use this to assign different IP addresses to different interfaces    
        * `ifconfig eth0 192.168.122.250`
        * however this is a temporary assignment that will not persist through a reboot

    * ##### `ifup etho0`
      * this command is used to bring up a specific network interface

    * ##### `ifdown eth0`
      * this command is used to bring down a specific network interface

    * ##### `route`
      * this utility will display the routing table
      * you can also add and delete routes
      * the current `ip route` command does the same thing but better
        * you can delete routes just like the newer command
        * `route del ROUTENAME`
      * `route add -net 192.168.10.0 netmask 255.255.255.0 gw 192.168.1.22.25`
        * this will add another route to your routing table

## 109.3 Basic Network Troubleshooting
### Testing Connectivity

  * ##### `ping`
    * this is used to test a system's ability to communicate with another network device
    * `ping` sends "echo" ICMP packets to see if the other system will respond
    * if yes, we know that there is general network connectivity between the two systems
    * if you do not get an echo reply, that means there is no connectivity between the two systems
    * if you get some packets and not others, then that means you have a spotty connection
    * `ping -c 3 198.127.12.1`
      * this will send three pings to the above IP address
    * `ping -6 -c 3 ::1`
      * this will send IPv6 packets over ICMP

  * ##### `traceroute`
    * this is used to trace the route that the packets travel to a destination
    * Typically you will travel:
      * from your local network
      * through your ISP
      * their connection to the internet
      * through the public web
      * then to the destination
    * this is by default sent by ICMP
    * you can change to TCP by using a -T flag on the command `traceroute -T IPADDRESS`
    * you can use IPv6 packets
      * traceroute6 or traceroute -6

  * ##### `tracepath`
    * a modern replacement for the traceroute command
    * this uses UDP packets instead of ICMP
    * `tracepath gooogle.com`
    * tracepath6 is a thing too

  * ##### `netstat`
    * this command is used to display every network connection and their state on a system
      * TCP, UDP, Sockets <= typical connections
    * this can also be used to view the routing table of a system
    * this is part of the `net-tools` package
    * `-l` - shows us connections that are listening
    * `-t` - shows us tcp connections
    * `-u` - shows us udp connections
    * `-p` - shows us the process id that is listening on those ports
    * `-r` - shows us the routing table

  * ##### `ss`
    * this is the modern equivalent of the `netstat` command
    * it is not able to show the routing table however
    * `ss` is short for "socket statistics" but it shows all the connections
    * luckily this command using many of the same switches as `netstat`


## 109.4 Configure Client-Side DNS
* ### Basic DNS Resolution
  * These are the basic configuration files to translate a hostname into an IP address
  * ##### `/etc/hosts`
    * this has been around for forever
    ```
    127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
    ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
    2600:3c03::f03c:91ff:fe9f:138d   nodeserver
    69.164.214.8   nodeserver
    ```
    * so this shows us our local loopback IP address
    * it also give us various hostnames we can use in place of the ip address
      * user@localhost4.localdomain4 or just user@localhost
      * localhost is the universal, default system hostname
    * we also can see an IPv6 version of  

  * ##### `/etc/hostname`
    * this is where the system prompt gets the hostname
    * this file is created or edited with the `hostnamectl` command
    * note that there is not an IP address listed to connect with the hostname
      * it is configured to connect with the regular network interface
      * you know it works by pinging the hostname


  * ##### `/etc/resolv.conf`
    * the IP addresses of the servers that our systems will contact for name resolution, reside in this file
    * this file contains a list of IP addresses that our system will use for DNS resolution
    * can have one to three IP addresses?

  * ##### `/etc/nsswitch.conf`
    * name service switch configuration file
    * this file is mostly used to determine how the order of name resolution occurs
    * on the left you have the database that is being queried for a particular service
    * on the right you have the files and sources the database uses to query, in top down order
      * for example lets look at the entry for hosts
      ```
      hosts:          files dns
      ```
      * when the computer needs to resolve a hostname to an ip address first it will look into files
        * files like `/etc/hosts` and `/etc/hostname`
      * then it will look into DNS with is handled by `/etc/resolv.conf` if it does not find the resolution in the local files
      * if you configured the service to look at DNS servers first and then look at the local files you may get a system performance hit
        * the reason is we would have to wait for the DNS query to timeout before we were able to query the local files
        * it is best to leave these settings alone unless you have  a very specific need or it never needs to communicate locally

  * ##### `host`
    * this is part of the bind-utils package
    * bind is standard nameserver package that is used throughout the internet
    * package contains utilities to make DNS queries thereby resolving domain names to IP addresses
    * Output of `host google.com` =>
    ```
    $ host google.com
    google.com has address 172.217.6.238
    google.com has IPv6 address 2607:f8b0:4006:811::200e
    google.com mail is handled by 20 alt1.aspmx.l.google.com.
    google.com mail is handled by 40 alt3.aspmx.l.google.com.
    google.com mail is handled by 30 alt2.aspmx.l.google.com.
    google.com mail is handled by 10 aspmx.l.google.com.
    google.com mail is handled by 50 alt4.aspmx.l.google.com.
    ```
      * output shows the first IP addresses (4 & 6) it found
      * also show mail servers
      * lastly the numbers of the mail servers indicate the priorty, lower number = higher priority
    * with all these answers from `hosts` you could hard code them into your `/etc/resolv.conf` but if something changes on the other end, then your configuration breaks

  * ##### `dig`
    * this command is used to query DNS servers for particular types of DNS records
    * at the bottom of the output, it shows which DNS sever was queried and the port
    * you can use a specific DNS resolver my indicating the ip address in the dig command
      * `dig @129.133.12.34 google.com`
    * you can use the `-t` option so that you can narrow down the records you would like
      * `dig -t A google.com`
      * `dig -t any google.com` will give you any and all records

  * ##### `getent`
    * this command will query `/etc/nsswitch.conf` and its corresponding database locations for information
