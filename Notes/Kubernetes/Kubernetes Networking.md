
Prerequisite: 
1. [Linux Network Namespaces](obsidian://open?vault=Notes&file=Linux%20Network%20Namespaces)
2. [Docker Networking](obsidian://open?vault=Notes&file=Docker%2FDocker%20Networking)


#### Check Ip range for pods in a cluster

``` Shell
kubectl logs -n kube-system weave-net-sr9nk weave | grep -Eo ".{0,1}ipalloc-range.{0,14}"
 ipalloc-range:10.244.0.0/16
```

- Check the logs of the weave contianer inside the weave-net-xxxx pods and look for the **ipalloc-range** keyword.
- Here the IP range is **10.244.0.0/16**

#### Check Ip range for services in the cluster

```Shell
controlplane ~ ➜  ps -aux | grep kube-apiserver | grep -E -o ".{0,2}service-cluster-ip-range.{0,14}"
--service-cluster-ip-range=10.96.0.0/12
```

- Check the runtime configuration for the `kube-apiserver` using the `ps -aux` command and grep the `--service-cluster-ip-range` keyword.
- Here the service ip range is `10.96.0.0/12`
