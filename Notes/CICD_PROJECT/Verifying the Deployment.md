
```
kubectl run curl --image=curlimages/curl -i --rm --restart=Never -- curl myjavaapp-myapp:8080
```

- Using this kubectl command we create a pod with curl image which curls the service at 8080 port and then the pod is deleted after it's work is done.
- If the exit code of this commad is 0 that means our application deployment was  a success else it was a failure

```Shell
stage("Verify application deployment on k8s-cluster") {
    steps {
        script{
            dir ("kubernetes/"){  
				sh 'kubectl run curl --image=curlimages/curl -i --rm --restart=Never -- curl myjavaapp-myapp:8080 ' 
			        }   
                }
            }
        } 
```

![[Pasted image 20230511231121.png]]