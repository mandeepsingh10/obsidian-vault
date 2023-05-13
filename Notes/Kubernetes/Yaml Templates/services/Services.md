# Services
## NodePort

This is used to map the port on which the webapp/application is running on the pod/container to the service port and then to the NodePort.

Range for NodePort is 30000-32767 which means a NodePort is unlikely to match a service's intended port (for example, 8080 may be exposed as 31020).

Services can be created just like pods, replicasets and deployments using a YAML definition file.

``` YAML
apiVersion: v1 
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30007
  selector:
    app: my-app

```
In the service ```myapp-service```, we have mapped ```port 80``` of  my-app application which is an nginx based application to the ```port 80``` of the ```NodePort service``` which is again mapped to the ```port 30007``` on the k8s node.

Now thw web application running on a pod on a k8s node can be accessed using the IP:port on the web browser, example - ```172.16.240.7:30007```
where 172.16.240.7 is the k8s node's IP address where the pod is running.

To create a service definded in a file **service.yaml**, we can use use the below mentioned kubectl command.

``` bash
kubectl create -f services.yaml
```
To list all the services on a k8s node, use the command
``` bash
kubectl get services
```
