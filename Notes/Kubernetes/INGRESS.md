
 - Ingress helps our users access our application using a single externally accessible URL that we can configure to route traffic to different services withing our cluster based on the URL path at the same time implement SSL security as well.
 - We can think of INGRESS as a Layer-7 Load balancer built in to the kubernetes cluster which can be configured using native k8s preemitives just like any other object that we've been working with in k8s.

```Shell
kubectl create ingress ingress-pay -n critical-space --rule=/pay*=pay-service:8282 --annotation nginx.ingress.kubernetes.io/rewrite-target=/ --annotation nginx.ingress.kubernetes.io/ssl-redirect="false" --dry-run=client -o yaml > ingress-pay2.yaml
```

- The above imperative command will create this yaml config file
``` YAML
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
  creationTimestamp: null
  name: ingress-pay
  namespace: critical-space
spec:
  rules:
  - http:
      paths:
      - backend:
          service:
            name: pay-service
            port:
              number: 8282
        path: /pay
        pathType: Prefix
status:
  loadBalancer: {}
```
