
- Whenever a service is created in K8s, the k8s DNS service creates a record for the service.  It maps the service name to the IP address.
- Suppose we have a two pods (in the same namespace, say default), `test-pod` and `web-pod`, we create a service `web-service` to access the `web-pod` from the `test-pod`, then the ip address and hostname of the `web-service` is stored in the DNS.
- Now the web-service can be accessed from the `test-pod` using only it's name `web-service` as they are in the same namespace. 

```Shell
curl http://web-service`
Welcome to NGINX!
```

#### What happens when the pod and the service are in different namespaces?

-  For each namespace the DNS server creates a subdomain.
- Let's say the `test-pod` is in default namespace and the `web-service` is in the apps namespace, then the `web-service` can be accessed from the `test-pod` using 
```Shell
curl http://web-service.apps
Welcome to NGINX!
```

- The last name `apps` of the service is the name of the `namespace` it is in.

##### All the services are grouped together into another subdomain called svc

- `web-service` is the name of the service. `apps` is the name of the namespace
- All the services are further grouped together into another subdomain called `svc`.

- So we can reach our application using 
```Shell
curl http://web-service.apps.svc
```

- Finally all the services and PODs are grouped together into a root domain for the cluster which is `cluster.local` by default.
- We can access the service using the URL `http://web-service.apps.svc.cluster.local`
 |Hostname | Namespace | Type | Root Domain | IP Address|
| --------- | --------- | --------- | --------- |--------|
| web-service      | apps    | svc      | cluster.local      | 10.107.37.188|

#### What about pods?

- Records for PODs are not created by default, but we can enable that explicitly.
- Once enabled, records are created for PODs as well, it does not use the POD name though.
- For each POD k8s generates a name by replacing the dots in the IP address to dashes - and the type is set to `pod`
| Hostname | Namespace | Type | Root Domain | IP Address|
| --------- | --------- | --------- | --------- |--------|
| 10-244-2-5      | apps    | pod      | cluster.local      | 10.244.2.5|

- The pod can be accessed using 
```Shell
curl http://10-244-2-5.apps.pod.cluster.local
Welcome to NGINX!
```
