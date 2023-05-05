
#### Control Plane Failures

1. Check status of the nodes
2. Check the status of the pods running in the cluster
3. If control plane components are deployed as pods then check their status
4. If control plane components are deployed as services then check the status of the services 
	   Master nodes - `kube-apiserver, kube-controller-manager, kube-scheduler` 
	   Worker nodes - `kubelet, kube-proxy`
5. Check logs of control plane components
	    kubeadm deployment - `kubectl logs -n kube-system kube-controller-manager-controlplane`
	    Hard way - `journalctl -u kube-apiserver`


#### Worker Node Failures

1. Check the status of the nodes
2. Check for different conditions in the Conditions section of node using the command
	   `kubectl describe node node1`
3. When a worker node stops communicating with the master node then the different flags under the condition section may be set to unknown, we can check the `LastHeartBeatTime` to check when the node crashed
4. Check resource consumption on node, status of kubelet service and logs
	    (`journalctl -u kubelet`)
5. Check the kubelet certificates that they are not expired, they are part of the right group and they are issued by the right CA.
	   `openssl x509 -in /var/lib/kubelet/worker-1.crt -text`