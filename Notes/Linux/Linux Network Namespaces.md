
#### List and create network namespaces

```Shell
~ ➜ sudo ip netns add red
~ ➜ sudo ip netns ls
red
```

#### Delete network namespaces

```shell
~ ➜ sudo ip netns delete red
~ ➜ sudo ip netns ls
~ ➜ 
```

#### Run commands within the network namespace

```shell
~ ➜ sudo ip netns exec red ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
12: eth0@if13: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever

```

- `ip a` command is executed inside the red network namespace
-  This can also be done using the command `ip -n red a `

- Now to establish connectivity between two network namespaces we need to create a virtual cable or pipe. 
- Suppose we want to create a virtual cable or pipe between two namespaces red and blue, this can be done like this
```Shell
~ ➜ sudo ip link add veth-red type veth peer name veth-blue
~ ➜ ip link | grep -e red -e blue -A2
14: veth-blue@veth-red: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 32:bf:7a:e0:4b:cc brd ff:ff:ff:ff:ff:ff
15: veth-red@veth-blue: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether a6:15:84:c7:42:c8 brd ff:ff:ff:ff:ff:ff

```

- Here the we specify the two ends of the cable **veth-red** and **veth-blue**
- Next we have to attach each interface to the appropriate network namespace, red end attaches to red network namespace and blue end attaches to ble network namespace, this is done in this way
```Shell
~ ➜ sudo ip link set veth-red netns red
~ ➜ sudo ip link set veth-blue netns blue
~ ➜ sudo ip -n red a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
15: veth-red@if14: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether a6:15:84:c7:42:c8 brd ff:ff:ff:ff:ff:ff link-netns blue
~ ➜ sudo ip -n blue a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
14: veth-blue@if15: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 32:bf:7a:e0:4b:cc brd ff:ff:ff:ff:ff:ff link-netns red

```

- Now we will assign ip addresses to the interfaces
```Shell
~ ➜ sudo ip -n red addr add 192.168.15.1/24 dev veth-red
~ ➜ sudo ip -n red a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
15: veth-red@if14: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether a6:15:84:c7:42:c8 brd ff:ff:ff:ff:ff:ff link-netns blue
    inet 192.168.15.1/24 scope global veth-red
       valid_lft forever preferred_lft forever
~ ➜ sudo ip -n blue addr add 192.168.15.2/24 dev veth-blue
~ ➜ sudo ip -n blue a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
14: veth-blue@if15: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 32:bf:7a:e0:4b:cc brd ff:ff:ff:ff:ff:ff link-netns red
    inet 192.168.15.2/24 scope global veth-blue
       valid_lft forever preferred_lft forever
```

- Let's bring up the interfaces
```Shell
~ ➜ sudo ip -n red link set veth-red up
~ ➜ sudo ip -n blue link set veth-blue up
~ ➜ sudo ip -n red a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
15: veth-red@if14: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether a6:15:84:c7:42:c8 brd ff:ff:ff:ff:ff:ff link-netns blue
    inet 192.168.15.1/24 scope global veth-red
       valid_lft forever preferred_lft forever
    inet6 fe80::a415:84ff:fec7:42c8/64 scope link 
       valid_lft forever preferred_lft forever
~ ➜ sudo ip -n blue a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
14: veth-blue@if15: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 32:bf:7a:e0:4b:cc brd ff:ff:ff:ff:ff:ff link-netns red
    inet 192.168.15.2/24 scope global veth-blue
       valid_lft forever preferred_lft forever
    inet6 fe80::30bf:7aff:fee0:4bcc/64 scope link 
       valid_lft forever preferred_lft forever
```

- Now let's check the connectivity between the two namespaces.
```Shell
~ ➜ sudo ip netns exec red ping -c3 192.168.15.2
PING 192.168.15.2 (192.168.15.2) 56(84) bytes of data.
64 bytes from 192.168.15.2: icmp_seq=1 ttl=64 time=0.027 ms
64 bytes from 192.168.15.2: icmp_seq=2 ttl=64 time=0.016 ms
64 bytes from 192.168.15.2: icmp_seq=3 ttl=64 time=0.015 ms

--- 192.168.15.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2047ms
rtt min/avg/max/mdev = 0.015/0.019/0.027/0.005 ms
~ ➜ sudo ip netns exec blue ping -c3 192.168.15.1
PING 192.168.15.1 (192.168.15.1) 56(84) bytes of data.
64 bytes from 192.168.15.1: icmp_seq=1 ttl=64 time=0.027 ms
64 bytes from 192.168.15.1: icmp_seq=2 ttl=64 time=0.015 ms
64 bytes from 192.168.15.1: icmp_seq=3 ttl=64 time=0.015 ms

--- 192.168.15.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2027ms
rtt min/avg/max/mdev = 0.015/0.019/0.027/0.005 ms
```

- The arp tables are also updated for both the namespaces
```Shell
~ ➜ sudo ip netns exec red arp
Address                  HWtype  HWaddress           Flags Mask       Iface
192.168.15.2             ether   32:bf:7a:e0:4b:cc   C                veth-red
~ ➜ sudo ip netns exec blue arp
Address                  HWtype  HWaddress           Flags Mask       Iface
192.168.15.1             ether   a6:15:84:c7:42:c8   C                veth-blue
```

- Also, the host has no idea about the namespaces that we have created.
```Shell
~ ➜ arp
Address                  HWtype  HWaddress           Flags Mask            Iface
192.168.31.27            ether   5c:02:14:61:e2:64   C                     eno2
XiaoQiang                ether   78:11:dc:17:ee:b9   C                     eno2
172.17.0.2               ether   02:42:ac:11:00:02   C                     docker0
```


### How do we connect more than two or say 100s of network namespaces together, how do we enable all of them to communicate with each other?

- Just like in the phyical world we create a virtual network inside a host,  to create a virtual network we need a **virtual switch** and then we **connect all the namespaces** to this **virtual switch**.
- We do this using the **Linux Bridge**, to create an internal bridge network we add a new interface to the host using the `ip link add` command
```Shell
~ ➜ sudo ip link add v-net-0 type bridge
~ ➜ ip link | grep -A2 v-net-0
16: v-net-0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 56:7b:c5:b0:22:d0 brd ff:ff:ff:ff:ff:ff
~ ➜ sudo ip link set dev v-net-0 up
```

- When we connected the two network interfaces red and blue earlier we created a virtual cable, now we will be connecting all the network namespaces to the switch so we don't need this virtual cable, let's delete it.
```Shell
sudo ip netns exec red ip link delete veth-red
~ ➜ sudo ip netns exec red ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
~ ➜ sudo ip netns exec blue ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```
- Deleting one end of the link also deletes the other end of the link as they are a pair

- Let's now create a new pair but this time veth-red is connected to the switch v-net-0, so the other end will be named veth-red-br.

```Shell
~ ➜ sudo ip link add veth-red type veth peer name veth-red-br
~ ➜ ip link | grep -A2 veth-red
17: veth-red-br@veth-red: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether de:a5:fc:ec:45:91 brd ff:ff:ff:ff:ff:ff
18: veth-red@veth-red-br: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether a6:15:84:c7:42:c8 brd ff:ff:ff:ff:ff:ff
```

-  Similarly we need to create a cable for blue namespace as well
```Shell
~ ➜ sudo ip link add veth-blue type veth peer name veth-blue-br
~ ➜ ip link | grep -A2 veth-blue
19: veth-blue-br@veth-blue: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 66:34:0c:d0:a0:37 brd ff:ff:ff:ff:ff:ff
20: veth-blue@veth-blue-br: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 32:bf:7a:e0:4b:cc brd ff:ff:ff:ff:ff:ff
```
- Now we need to connect it to the bridge network, to attach one end of the interface to the red network namespace run
```Shell
sudo ip link set veth-red netns red
```
- Now to connect the other end to the virtual switch (bridge network) run the command
```Shell
sudo ip link set veth-red-br master v-net-0
```
- Let's check the connection details
```Shell
~ ➜ ip link | grep  -A1 veth-red
17: veth-red-br@if18: <BROADCAST,MULTICAST> mtu 1500 qdisc noop master v-net-0 state DOWN mode DEFAULT group default qlen 1000
    link/ether de:a5:fc:ec:45:91 brd ff:ff:ff:ff:ff:ff link-netns red
```
- Similarly we do the same steps to connect the blue namespace with the bridge network
```Shell
~ ➜ sudo ip link set veth-blue netns blue
~ ➜ sudo ip link set veth-blue-br master v-net-0
~ ➜ ip link | grep -A1 veth-blue
19: veth-blue-br@if20: <BROADCAST,MULTICAST> mtu 1500 qdisc noop master v-net-0 state DOWN mode DEFAULT group default qlen 1000
    link/ether 66:34:0c:d0:a0:37 brd ff:ff:ff:ff:ff:ff link-netns blue
```

- Now we will assign the ip addresses to these networks in the respective namespaces and will try to bring them up
```Shell
~ ➜ sudo ip -n red addr add 192.168.15.1/24 dev veth-red
~ ➜ sudo ip -n blue addr add 192.168.15.2/24 dev veth-blue
~ ➜ sudo ip -n red link set dev veth-red up
~ ➜ sudo ip -n blue link set dev veth-blue up
```

- Let's check the network details of the bridge network as well the red and blue network namespaces
```Shell
~ ➜ ip link | grep -A2 -e veth-red -e veth-blue
27: veth-red-br@if28: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master v-net-0 state UP mode DEFAULT group default qlen 1000
    link/ether de:a5:fc:ec:45:91 brd ff:ff:ff:ff:ff:ff link-netns red
29: veth-blue-br@if30: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master v-net-0 state UP mode DEFAULT group default qlen 1000
    link/ether 66:34:0c:d0:a0:37 brd ff:ff:ff:ff:ff:ff link-netns blue
~ ➜ sudo ip netns exec red ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
28: veth-red@if27: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether a6:15:84:c7:42:c8 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.15.1/24 scope global veth-red
       valid_lft forever preferred_lft forever
    inet6 fe80::a415:84ff:fec7:42c8/64 scope link 
       valid_lft forever preferred_lft forever
~ ➜ sudo ip netns exec blue ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
30: veth-blue@if29: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 32:bf:7a:e0:4b:cc brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.15.2/24 scope global veth-blue
       valid_lft forever preferred_lft forever
    inet6 fe80::30bf:7aff:fee0:4bcc/64 scope link 
       valid_lft forever preferred_lft forever
```

- The virtual interfaces at the two ends of the virtual cables can be  identified using their numbers, odd and even forms a pair, here **veth-red@if17** end of the cable is connected to **veth-red-br@if18** end of the bridge network
- Similarly **veth-blue@if19** on the blue network namespace is connected to **veth-blue-br@if20** on the bridge network.

- If the network namespaces are not able to ping each other then check the iptables rules
```Shell
~ ➜ sudo iptables --list FORWARD
Chain FORWARD (policy ACCEPT) 
target     prot opt source               destination 
```
The iptables FORWARD chain policy should accept packets

- Now if we try to access the 192.168.15.1 ip from our host, we won't be able to reach it because the host is on one network and the namespace is on another.

#### What if we want to establish connectivity between the namespaces and the host?

- Remember that the bridge switch is actually a network interface so we just need to assign an ip address to it
```Shell
~ ➜ sudo ip addr add 192.168.15.5/24 dev v-net-0
~ ➜ ping -c2 192.168.15.1
PING 192.168.15.1 (192.168.15.1) 56(84) bytes of data.
64 bytes from 192.168.15.1: icmp_seq=1 ttl=64 time=0.024 ms
64 bytes from 192.168.15.1: icmp_seq=2 ttl=64 time=0.047 ms

--- 192.168.15.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1035ms
rtt min/avg/max/mdev = 0.024/0.035/0.047/0.011 ms
```

- We can now ping our namespaces from our host

### What happens when we try to reach another network say 192.168.1.3 from the network namespaces?

```Shell
sudo ip netns exec red ping 192.168.1.3
ping: connect: Network is unreachable
```

- For this to work we need to add an entry to the routing table to provide a gateway to the outside world.
- The 192.168.1.3 network is connected to our host and we are connected to the host using the bridge network which is **192.168.15.5**, so we need to add a route in the routing table to route all requests where the destination is 192.168.1.0/24 to 192.168.15.5
```Shell
sudo ip netns exec red ip route add 192.168.1.0/24 via 192.168.15.5
sudo ip netns exec red route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.15.0    0.0.0.0         255.255.255.0   U     0      0        0 veth-red
192.168.1.0    192.168.15.5    255.255.255.0   UG    0      0        0 veth-red
```

- Now when we try to ping the 192.168.1.3 IP, we don't get the network not reachable error but we are still not able to ping it.
```Shell
~ ➜ sudo ip netns exec red ping 192.168.31.7
PING 192.168.31.7 (192.168.31.7) 56(84) bytes of data
```

- This is because our home network has our internal private IP addresses that the destination network doesn't know about so they cannot reach back.
- We need to make it look like the request it coming from the host network for the destination network to send a response, for this we need NAT enabled on our host acting as a gateway here so that it can send the messages to the LAN in it's own name with it's own addresses.

#### How do we add NAT functionality to our host?

- We do that using iptables, add a new rule in the **NAT iptable** in the **POSTROUTING** chain to **MASQUERADE** or replace the **from address** on all the packets coming from the source network**192.168.15.0/24** with it's own IP address. 
```Shell
sudo iptables -t nat -A POSTROUTING -s 192.168.15.0/24 -j MASQUERADE
```
- In this way, anyone receiving these packets outside the network will think that they are coming from the host and not from the network namespaces, when we try to ping now we will get the response. 

#### How do we connect these namespaces to the internet?

- When we try to ping a server on the internet say 8.8.8.8 from the red namespace, we are not able to ping it and it says **Network is unreachable**
```Shell
sudo ip netns exec red ping 8.8.8.8
ping: connect: Network is unreachable
```

- Now we know what is the reason behind this, we check the routing table and see that there are no routes for the destination 8.8.8.8, now since these namespaces can reach any network that our host can reach, we can add a default route saying that route all external traffic to the host
```Shell
sudo ip netns exec red ip route add default via 192.168.15.5
```

- Now we should be able to reach the outside world from the network namespaces.
```Shell
~ ➜ sudo ip netns exec red ping -c2 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=118 time=9.66 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=118 time=10.9 ms

--- 8.8.8.8 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 9.660/10.284/10.909/0.624 ms
```

####  What about the connectivity from the outside world to web application in the network namespaces?

- As of now, the namespaces are on an internal private network and no one knows about them.
- Say we want to connect to an web application in red network namespace from the 192.168.1.3 computer.
- One way is to use NAT and add an iptable rule on the 192.168.1.3 computer telling that the network 192.168.15.0 can be reached using the 192.168.1.2 computer.
- Second way is to use port forwarding, we add a port forwarding rule using iptables saying that any traffic on port 80 of the host 192.168.1.2 should be redirected to the port 80 on the red namespace.
```Shell
sudo iptables -t nat -A PREROUTING --dport 80 --to-destination 192.168.15.2:80 -j DNAT
```
- docker uses this method to map host port to the container port.

