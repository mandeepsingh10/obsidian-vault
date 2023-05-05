Long commands like loops

```Shell
command: ["/bin/sh", "-c", "while true; do echo hello; sleep 10;done"]
```

```Shell
kubectl auth can-i get pods  --as system:serviceaccount:default:pink-sa-cka24-arch
```


1. Find the node across all clusters that consumes the most CPU and store the result to the file `/opt/high_cpu_node` in the following format `cluster_name,node_name`.

```Shell
kubectl top node --context cluster1 --no-headers | sort -nr -k2 | head -1
kubectl top node --context cluster2 --no-headers | sort -nr -k2 | head -1
kubectl top node --context cluster3 --no-headers | sort -nr -k2 | head -1
kubectl top node --context cluster4 --no-headers | sort -nr -k2 | head -1
```

2. 