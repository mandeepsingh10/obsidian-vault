
#### 1. Let's start by checking if we have any release created on our cluster

```Shell
jenkins@jenkins:~$ helm list
NAME     	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
```
- We don't have any helm releases

#### 2. Let's add a new stage to our jenkins pipeline to create a release

```Shell
stage('Deploying application on k8s-cluster') {
    steps {
        script{
            dir ("kubernetes/"){  
				sh 'helm upgrade --install --set image.repository="15.206.89.243:8083/springapp" --set image.tag="${VERSION}" myjavaapp myapp/ ' 
			        }   
                }
            }
        }
```

The `steps` block contains a `script` block that changes the working directory to `kubernetes/` using the `dir` directive. This is where the Helm chart and the Kubernetes deployment configuration files are stored.

The command is a Helm CLI command that upgrades an existing Kubernetes deployment or installs a new one using a Helm chart located in the `myapp/` directory.

The command is broken down as follows:

-   `helm upgrade`: This command upgrades a Helm release if it already exists, or installs a new one if it does not exist.
-   `--install`: This flag indicates that a new release should be installed if it does not already exist.
-   `--set`: This flag sets one or more values in the Helm chart's values file. In this case, it sets the `image.repository` and `image.tag` values to the specified values.
-   `image.repository="15.206.89.243:8083/springapp"`: This sets the Docker image repository for the application to `15.206.89.243:8083/springapp`.
-   `image.tag="${VERSION}"`: This sets the Docker image tag for the application to a value stored in the `${VERSION}` environment variable defined in the pipeline
-   `myjavaapp`: This is the name of the Helm release.
-   `myapp/`: This is the path to the Helm chart directory containing the `values.yaml` file and any other necessary files.

#### 3. We had created a secret for authenticating to the nexus repo while integrating jenkins & nexus with the k8s cluster

- When we run the command mentioned in step2,  the image.tag will get the value of $VERSION variable that is build number and it will also be replaced in the `values.yaml` file in helm charts directory
```YAML
# Default values for myapp.

# This is a YAML-formatted file.

# Declare variables to be passed into your templates.

  

replicaCount: 2

  

image:

	repository: IMAGE_NAME
	
	### this is image.repository, it will be replaced by nexus_ip:8083/springapp
	
	pullPolicy: IfNotPresent
	
	# Overrides the image tag whose default is the chart appVersion.
	
	tag: IMAGE_TAG 
	#### this is image.tag, it will be replaced by build number of the pipeline

  

service:

	type: NodePort
	
	port: 8080
  
```  

- Now when the helm charts command `helm upgrade --install --set image.repository="15.206.89.243:8083/springapp" --set image.tag="${VERSION}" myjavaapp myapp/` will be excuted it will replace all the values of variables, labels, annotation from the helm helper, chart.yaml and values.yaml in the deployment.yaml and services.yaml
- In deployment.yaml, in the pod template section, we have a field called `imagePullSecrets`,
   this field indicates that the container images for the pods are stored in a private repository.
```YAML
template:
    metadata:
      labels:
        {{- include "myapp.selectorLabels" . | nindent 8 }}
    spec:
      imagePullSecrets:
        - name: registry-secret
```

- The secret `registry-secret` is the same secret that we created.

#### 4. Let's start the build for our pipeline

	![[Pasted image 20230511215440.png]]

- Build was successfull, let's check the status of the deployment on the k8s-cluster

#### 5. Check the deployment and services created on the k8s-cluster

```Shell
root@k8s-master:~# kubectl get deployments
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
myjavaapp-myapp   2/2     2            2           95m
root@k8s-master:~# kubectl get svc
NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes        ClusterIP   10.96.0.1       <none>        443/TCP          147m
myjavaapp-myapp   NodePort    10.97.223.171   <none>        8080:30484/TCP   95m
root@k8s-master:~# kubectl get pods -o wide
NAME                               READY   STATUS    RESTARTS   AGE   IP          NODE        NOMINATED NODE   READINESS GATES
myjavaapp-myapp-76d89db84c-gzz5d   1/1     Running   0          77m   10.44.0.3   k8s-node1   <none>           <none>
myjavaapp-myapp-76d89db84c-tld87   1/1     Running   0          76m   10.44.0.2   k8s-node1   <none>           <none>
```
```Shell
jenkins@jenkins:~$ helm list
NAME     	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
myjavaapp	default  	2       	2023-05-11 15:14:23.505314623 +0000 UTC	deployed	myapp-0.2.0	1.16.0  
```

#### 6. Access the web app 

- We can use the Ip address of any one of the nodes with nodePort: 30484 `http://65.0.45.72:30484/`

	![[Pasted image 20230511220047.png]]

- Our Java web application is up and running.