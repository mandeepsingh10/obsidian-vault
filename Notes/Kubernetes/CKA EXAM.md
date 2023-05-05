#### Question 1

1. Create a new Clusterrole name **deployment-clusterrole**, which only allows to create the following resources types - **deployment,statefulSet,Daemonset** .
2. Create a new Serviceaccount named **cicd-token** in the existing namespace **app-team1** bind the new clusterrole **deployment-clusterrole** to the new service account **cicd-token** limited to the namespace **app-team1**

#### Solution

```Shell
kubectl create clusterrole deployment-clusterrole --verb=create --resource=Deployment,StatefulSet,DaemonSet

kubectl create sa cicd-tocken --namespace=app-team1  

kubectl create clusterrolebinding deployment-bind --clusterrole=deployment-clusterrole --serviceaccount=app-team1:cicd-token
```

#### Question 2

Set a node named eks-node-0 as unavailale and reschedule all the pods running on it.

#### Solution

```Shell
kubectl drain eks8-node0  --ignore-daemonsets --force --delete-local-data
```

#### Question 3

Given an existing kubernetes cluster running version 1.18.8, upgrade all of the kubernetes control plane and node components on the master node only to version 1.19.0  
You are expected to upgrade kubelet and kubectl on the master node.

#### Solution

```Shell
kubectl drain mk8s-master-0 --ignore-daemonsets  (if needed add --force --delete-local-data)

kubectl get nodes  

ssh mk8s-master-0
apt-get update  
apt-get install -y kubeadm=1.19.0-00  
kubeadm upgrade plan  
kubeadm upgrade apply v1.19.0  
kubeadm version

sudo kubeadm upgrade node
sudo kubeadm upgrade app

```

#### Question 4

Create a new network policy named **allow-port-from-namespace** that allows pods in the existing namespace **my-app** to connect to **port 9000** of other pods in the same namespace.  
Ensure that the new NetworkPolicy
-  does not allow access to pods not listening on port 9000  
- does not allow access from Pods not in namespace my-app

#### Solution

``` yaml 
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy  
metadata:  
  name: tallow-port-from-namespace  
  namespace: my-app  
spec:  
  podSelector: {}  
  
  policyTypes:  
  - Ingress  
  
  ingress:  
  - from:  
  
    - namespaceSelector:  
        matchLabels:  
          project: my-app  
  
    ports:  
    - protocol: TCP  
      port: 9000
```



#### Question 5

Create a snapeshot of the existing etcd instance running at [https://127.0.0.1:2379](https://127.0.0.1:2379/), saving the snapshot to **/srv/data/etcd-snapshot.db**  
Next restore an existing, previous snapshot located at **/var/lib/backup/etcd-snapshot-previous.db**  
CA cert: /op/KUNIN00601/ca.crt  
Client crt /op/KUNIN00601/etcd-client.crt  
Client key /op/KUNIN00601/etcd-client.key

#### Solution

```Shell
ETCDCTL_API=3 etcdctl --endpoints https://127.0.0.1:2379
--cacert=/op/KUNIN00601/ca.crt \ 
--cert=/op/KUNIN00601/etcd-client.crt \
--key=/op/KUNIN00601/etcd-client.key \
snapshot save /srv/data/etcd-snapshot.db

# verify the snapshot  
ETCDCTL_API=3 etcdctl --write-out=table \
snapshot status /srrv/data/etcd-snapshot.db

```

#### Question 6

Reconfigure the existing deployment **front-end** and add a port specification named **http** exposing **port 80/tcp** of the existing container **nginx**  
Create a new service named **front-end-svc** exposing the container **port http**.  
Configure the service also expose the individual pods via a **NodePort** on the nodes on which they are scheduled.

#### Solution

```Shell
kubectl edit deployment.apps front-end

#Go to container spec > add the following  under the name: nginx  
  containers:  
  - image: nginx:1.14.2  
    name: nginx  
    ports:  
    - containerPort: 80  
      name: "http"  
      protocol: TCP
 
 ####     
 kubectl expose deployment front-end --name=front-end-svc --port=80 --type-NodePort --protocol=TCP
```

#### Question 7 

Create a new nginx Ingress resource as follows:  
  - Name: pong  
  - Namespace: in-internal  
  - Exposing service service hello on path /hello using service port 5678  
  - Availablity of sevice can be checked by curl -kL ip/hello

#### Solution

```Shell
apiVersion: networking.k8s.io/v1  
kind: Ingress  
metadata:  
  name: pong  
  namespace: in-internal   
spec:  
  rules:  
  - http:  
      paths:  
      - path: /hello  
        pathType: Prefix  
        backend:  
          service:  
            name: hello  
            port:  
              number: 5678  

####
kubectl create -f ingress.yaml

####
kubectl --namespace=in-internel describe ingress pong
```

#### Question 8

Scale the deployment presentaion to 6 pods.

#### Solution

```Shell
kubectl scale deployment presentaion --replicas=6
```

#### Question 9 

Schedule a pod as follows:  
- Name: nginx-kusc00401  
- Image: nginx  
- Node Selector: disk=ssd

#### Solution 

```Shell
kubectl run nginx-kusc00401 --image --dry-run -o yaml > kuse.yaml
```
```YAML
apiVersion: v1  
kind: Pod  
metadata:  
  name: nginx  
  labels:  
    run: nginx-kusc00401  
spec:  
  containers:  
  - name: nginx  
    image: nginx  
  nodeSelector:  
    disktype: ssd
```
```Shell
kubectl create -f kuse.yaml
```

#### Question 10

Check to see how many nodes are ready (not including nodes tainted NoSchedule) and write the number to /opt/KUSC00402/kusc00402.txt

#### Solution 

```Shell
kubectl get nodes echo "2" > /opt/KUSC00402/kusc00402.txt
```

#### Question 11

Create a pod named kucc4 with single app container for each of the following images running inside (there may be between q and 4 images specified):  
nginx + redis + memcached + consul

#### Solution

```Shell
kubectl run kucc4 --image-nginx --dry-run -o yaml > 4.yaml
```
```YAML
apiVersion: batch/v1  
kind: pod  
metadata:  
  labels:  
     run: kucc4  
  name: kucc4  
spec:  
  containers:  
  - name: nginx  
    image: nginx  
  - name: redis  
    image: redis  
  - name: memcashed  
    image: memcashed  
  - name: consul  
    image: consul
```

#### Question 12

Create a persistent volume with name **app-data,** of **capacity 1Gi** and **access mode ReadWriteMany**, The type of volume is **hostPath** and its location is **/srv/app-data**

#### Solution

```YAML
apiVersion: v1  
kind: PersistentVolume  
metadata:  
  name: app-data  
spec:  
  storageClassName: manual  
  capacity:  
    storage: 1Gi  
  accessModes:  
    - ReadWriteMany  
  hostPath:  
    path: "/srv/app-data"
```

#### Question 13

Create a new PersistentVolumeClaim:
- Name: pv-volume.  
- Class:  csi-hostpath-sc  
- Capacity: 10Mi\

Create pod and claim the space from PV
#### Solution

```YAML
apiVersion: v1  
kind: PersistentVolumeClaim  
metadata:  
  name:pv-volume  
spec:  
  storageClassName: csi-hostpath-sc  
  accessModes:  
    - ReadWriteOnce  
  resources:  
    requests:  
      storage: 10Mi
```
```Shell
kubectl create -f pv-claim.yaml  
kubectl get pvc
```

#### Question 14

Monitor the logs of pod foo and:  
- Extract log lines corrosponding to error " unable-to-access-website"  
- Write them to /opt/KUTR00101/f00

#### Solution 

```Shell
kubectl logs foo | grep unable-to-access-website > /opt/KUTR00101/f00  
cat /opt/KUTR00101/f00
```

#### Question 15 

Without changing its existing containers, an existing Pod needs to be integrated into Kubernetes's built-in logging architecture (eg: kubectl logs). Adding a steaming sidecar container is a good and common way to accomplish this requirment.  

Task: Add a busybox sidecar container to the existing Pod 11-factor-app. The new sidecar container had to run the following command :  
`/bin/sh -c tail -n+1 /var/log/11-factor-app.log`

Use a volume mount named logs to make the file /var/log/11-factor-app.log available to the sidecar container.  
- Don't modify the existing container.
- Don't modify the path of the log file.
- Both container must be access it at containers must access it at /var/log/110factor-app.log.

#### Solution

- Similar to the question in Kodekloud CKA ULTIMATE MOCK SERIES


#### Question 16

Find the pod with label **name=cpu-utilizer**, running high CPU workloads and write the name of the pod consuming most to the file /opt/KUTR00401/KUTROO401.txt (Which is already exist)

#### Solution 

```Shell
kubectl top pods -l name=cpu-utilizer  

kubectl top pods -l name=cpu-utilizer --sort-by=cpu --no-headers | cut -f1 -d" "" | head -n1 > /opt/KUTR00401/KUTROO401.txt
```

#### Question 17

A kubernetes worker node, named wk8s-node-0 is in state NotReady. Investigate why this is the case, and perform  any appropriate steps to bring the node to a Ready state, ensureing that any changes are made permanent. You can assume elevated privileages sudo -i

#### Solution

```Shell
#### ssh to node01  
systemctl enable kubelet  or systemctl enable --now kubelet  
systemctl restart kubelet  
systemctl status kubelet
```

