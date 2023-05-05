### Prerequisites
1. Servers needed for the cluster should be provisioned
2. All the nodes in the cluster should resolve each other using their hostname.
      - We have two nodes in our cluster
	  1. k8s-master - `172.31.10.145/20`
	  2. k8s-node1 - `172.31.2.234/20`
      - Add entries for all the nodes in /etc/hosts file of each node
```Shell
printf "\n172.31.10.145 k8s-master\n172.31.2.234 k8s-node1\n\n" >> /etc/hosts
```


### STEPS -- ALL NODES

###### 1. Loading modules and forwarding IPv4 and letting iptables see bridged traffic

  ```Shell
  printf "overlay\nbr_netfilter\n" >> /etc/modules-load.d/k8s.conf
  ```
	This loads the `overlay` and `br-netfilter` driver and it persists after reboot.
	`modprobe overlay` and `modprobe br_netfilter` loads the modules instantly.
		
-  Now let's enable IPv4 Forwarding and letting iptables see bridged traffic.
  These are sysctl params required by setup, params should persist across reboots.
```Shell
printf "net.bridge.bridge-nf-call-iptables = 1\nnet.ipv4.ip_forward = 1\nnet.bridge.bridge-nf-call-ip6tables = 1\n" >> /etc/sysctl.d/k8s.conf
```
- Apply sysctl params without reboot - `sudo sysctl --system`
- These parameters determine [whether packets crossing a bridge are sent to `iptables` for processing](https://wiki.libvirt.org/page/Net.bridge.bridge-nf-call_and_sysctl.conf). Most Kubernetes CNIs rely on `iptables`, so this is usually necessary for Kubernetes.

- Verify that the `br_netfilter`, `overlay` modules are loaded by running below instructions:
```Shell
	
```