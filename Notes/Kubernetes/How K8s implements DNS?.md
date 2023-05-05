
- The DNS implemented by K8s is known as CoreDNS, earlier prior to v1.12 it was known as kube-dns.

#### How CoreDNS is Setup in the cluster?

- The CoreDNS server is deployed as pods which are part of a replicaset, which in turn are again part of a deployment in the kube-system namespace in the k8s cluster.
- These PODs runs the CoreDNS executable.
- CoreDNS requires a configuration file, k8s uses a file calleed Corefile present in `/etc/coredns/Corefile`
```Shell
cat /etc/coredns/Corefile
:53 {
    errors
    health 
    kubernetes cluster.local in-addr.arpa ip6.arpa {
       pods insecure
       upstream
       fallthrough in-addr.arpa ip6.arpa
    }
    prometheus :9153
    proxy . /etc/resolv.conf 
    cache 30
    loop
    reload
}
```

- #### This Corefile is passed to the POD as a configMap object.
```Shell
controlplane ~ ➜  k describe configmap -n kube-system coredns 
Name:         coredns
Namespace:    kube-system
Labels:       <none>
Annotations:  <none>

Data
====
Corefile:
----
.:53 {
    errors
    health {
       lameduck 5s
    }
    ready
    kubernetes cluster.local in-addr.arpa ip6.arpa {
       pods insecure
       fallthrough in-addr.arpa ip6.arpa
       ttl 30
    }
    prometheus :9153
    forward . /etc/resolv.conf {
       max_concurrent 1000
    }
    cache 30
    loop
    reload
    loadbalance
}


BinaryData
====

Events:  <none>

```
- Within this file we have a number of plugins configured, the plugins in the above snippet are `errors`, `health`, `kubernetes`, `prometheus`, `proxy`, `cache`, `reload`
- These plugins are configured for handling errors, reporting health, monitoring metrics, cache etc.
- The plugin that make CoreDNS works with k8s is the `kubernetes` plugin, and this is where the top level domain name for the cluster is set, in this case `cluster.local` which is the default.
- Every record in the CoreDNS server falls under this domain `cluster.local`
- With `kubernetes` plugin we have multiple options, the `pods` option that we see here is responsible for creating a record for PODs in the cluster (addinng record for pod by replcaing the . in the ip address with a -), it is disabled by default.
- Any record that this DNS server can't resolve, for example say a POD tries to reach `www.google.com` it is forwarded to the nameserver specified in the coredns pods `/etc/resolv.conf` file.
- `/etc/resolv.conf` file is set to use the nameserver from the kubernetes node.

- The CoreDNS pod watches the k8s cluster for new PODs or services and everytime a pod or a service is created it adds a record for it in it's database.

#### How does the PODs in the cluster points/ try to reach the CoreDNS server, what address do they use?

- When we deploy the CoreDNS solution, it also creates a service to make it available to other components within the cluster.
- The service is named as `kube-dns` by default. The IP address of this service is configured as the nameserver on the PODs, this is done by k8s automatically when a POD is created, the `kubelet` component is responsible for it.
- If we look at the config file of the `kubelet` we will see the IP address of the DNS server and the domain name in it.
```Shell
cat /var/lib/kubelet/config.yaml  | grep clusterDNS -A2
clusterDNS:
- 10.96.0.10
clusterDomain: cluster.local
```

- Once the PODs are configured with the right nameserver, we can now resolve other PODs and services
- The `/etc/resolv.conf` file in the pod has the following entries.
```
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
```
- It also has a search entry which helps in finding a service using any name, `web-service` or `web-service.apps` or `web-service.apps.svc`
- It only has seacrch entries of the services and not the PODs so we won't be able to reach the PODs in the same way, for that we need to specify the FQDN of the POD like 
  `10-244-2-5.apps.pod.cluster.local`

- To check a pod the name resolution of a service on another namespace use `nslookup` or `host` command
```Shell
kubectl exec -it hr -- host mysql.payroll 
mysql.payroll.svc.cluster.local has address 10.101.6.222

kubectl exec -it hr -- nslookup mysql.payroll 
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   mysql.payroll.svc.cluster.local
Address: 10.101.6.222

```
- Here we run the `nslookup` and `host` command on the `hr` POD in the `default` namespace to check name resolution of the `mysql` POD in the `payroll` namespace.