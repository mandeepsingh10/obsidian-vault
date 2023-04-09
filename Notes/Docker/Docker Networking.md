
### Types of network drivers
1. none
2. host
3. bridge
4. overlay
5. ipvlan
6. macvlan

Refer: [Network drivers](https://docs.docker.com/network/)
  
- `docker network ls`  command lists the available networks in docker environment.

``` shell
~ ➜ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
5fb5ec1f2af4   bridge    bridge    local
dd550ff4afa7   host      host      local
a0b1f673ee87   none      null      local
```

#### Bridge Network

- The `bridge` network is nothing but a bridge interface created by docker on the host.
```shell
docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:a3:3a:83:69 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:a3ff:fe3a:8369/64 scope link 
```

- It is just like adding a new interface using the `ip link add docker0 type bridge` command.
- `docker0` is an like an interface for the host but it acts as a switch for the docker containers.
- IP address of the `docker0` interface on the host is `172.17.0.1/16` 
- Whenever a docker container is created docker creates a network namespace for that container. 
- The network namespaces be checked using the `ip netns` command but by default it will not list the docker network namespace as the `ip netns` command looks in the `/var/run/netns` path for the namespace but docker network namespaces are present in `/var/run/docker/netns` so we'll need a bind mount for these two directories to be able to view the docker network namespaces using the `ip netns command` .
```shell
~ ➜ sudo ls /var/run/docker/netns
07c0e3775d82
```

- We can see the namespace connected with each container using the `docker inspect` command, look for  `SandboxKey` object. 
  Example: `SandboxKey": "/var/run/docker/netns/07c0e3775d82`
```shell
~ ➜ docker ps 
CONTAINER ID   IMAGE               COMMAND              CREATED          STATUS          PORTS                                   NAMES
6cd8c1d44105   msx9797/webserver   "httpd-foreground"   53 minutes ago   Up 53 minutes   0.0.0.0:7900->80/tcp, :::7900->80/tcp   mywebsite
~ ➜ docker inspect mywebsite | grep NetworkSettings -A20
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "07c0e3775d821c37f9bb8193fd1e7d213234f800513bb1c092ddff70483b24d4",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {
                "80/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "7900"
                    },
                    {
                        "HostIp": "::",
                        "HostPort": "7900"
                    }
                ]
            },
            "SandboxKey": "/var/run/docker/netns/07c0e3775d82",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,

```

##### How does docker attach the container or it's network namespace to the bridge network?

- We know that the `docker0` interface acts as a switch for the docker containers.
- It creates a cable, a VIRTUAL cable, with two interfaces on each end.
- `docker inspect -f '{{.State.Pid}}' 6cd8c1d44105` gives the pid of the docker container and then `nsenter -t 4360 -n ip a` command gives the network details of the docker container.
```Shell
~ ➜ docker inspect -f '{{.State.Pid}}' 6cd8c1d44105
4360
~ ➜ sudo nsenter -t 4360 -n ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
8: eth0@if9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever

```

![[docker_networking.png]]
```shell
~ ➜ ip a | grep docker
5: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
9: veth9e3981e@if8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 

```


- The virtual interfaces at the two ends of the virtual cables can be  identified using their numbers, odd and even forms a pair, 8&9 are one pair, 10&11 forms another pair.
- In the image above the pairs are **veth9e3981e@if8** & **eth0@if9** .


#### What happens when we try to access the application running on a docker container?

- Suppose we have a nginx application running on the container on port 80, then we can access the application using the ip address of the container and the port like 
  curl `http://172.17.0.3:80`.
```Shell
~ ➜ curl -I http://172.17.0.2:80
HTTP/1.1 200 OK
Date: Sun, 09 Apr 2023 03:46:19 GMT
Server: Apache/2.4.55 (Unix)
Last-Modified: Sat, 11 Feb 2023 18:27:06 GMT
ETag: "5b2-5f470c2d7f280"
Accept-Ranges: bytes
Content-Length: 1458
Content-Type: text/html
```

- The application is reachable from the docker container. 
- Now let's try to connect to the application from outside the docker host.
```Shell
curl -I http://172.17.0.2:80
curl: (7) Failed to connect to 172.17.0.2 port 80 after 3068 ms: No route to host
```
- We cannot access the application from outside the docker host so to allow the access to the application running on the docker container for external users, docker provides a port mapping option.
```shell
~ ➜ docker run -itd --name=mywebsite -p 7900:80 msx9797/webserver
fc318afbd64ddcb35efd585dfbcd8ca5f336ab67b8ad61eb7330f2a482e5090a
```
- Here we map the port 80 of the docker container to port 7900 of the host, so now external users can access the application using the host IP and port combination like
  `192.168.1.10:7900` . Any traffic on the docker host on port 7900 will be forwarded to the docker container on port 80.

### But how does docker do this?

- The requirement is to forward traffic coming in on port to another port on the docker container. 
- We create a NAT rule for that, using iptables we create an entry into the NAT table, to append a rule to the PREROUTING chain to change the destination port from 7900 on host port to port 80 on the docker container.
- Docker also uses the same approach, it adds a rule to the DOCKER chain and sets the destination to include the containers IP as well.
- We can see this rule using the iptables command
```shell
~ ➜ sudo iptables -nL DOCKER -t nat 
Chain DOCKER (2 references)
target     prot opt source       destination         
RETURN     all  --  0.0.0.0/0    0.0.0.0/0           
DNAT       tcp  --  0.0.0.0/0    0.0.0.0/0      tcp dpt:7900 to:172.17.0.2:80
```


