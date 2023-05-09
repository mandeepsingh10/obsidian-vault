
1. Create a repository in nexus for Helm Charts
	Goto Nexus repo dashboard  > `Repositories` > `Create Repository`
	Select Recipe as `helm hosted`
	![[Pasted image 20230509185904.png]]

2. We can see our `helm-hosted` repository listed under repositories.
	![[Pasted image 20230509185950.png]]

3. Now let's create the Stage in Jenkins Pipeline for pushing the helm charts to nexus repo.
	
	No other configuration is need in Jenkins host because we will use nexus repo api to push the helm charts.
```
 curl -u admin:$nexus_password http://nexus_machine_ip:8081/repository/helm-hosted/ --upload-file myapp-${helmchartversion}.tgz -v	
```

   In our case the variables are
     - nexus_password   -> Variable we created in jenkins `nexus_docker_repo_pass_var` 
     - nexus_machine_ip -> `52.66.172.21`

   So the api command will look like this.
	   `curl -u admin:$nexus_docker_repo_pass_var http://52.66.172.21:8081/repository/helm-hosted/ --upload-file myapp-${helmversion}.tgz -v`

  For `helmchartversion` we will helm chart version for our application.
```Shell
->helm show chart kubernetes/myapp/
apiVersion: v2
appVersion: 1.16.0
description: A Helm chart for Kubernetes
name: myapp
type: application
version: 0.2.0

```
```
-> helm show chart myapp/ | grep version | awk '{print $2}'
0.2.0
```


The Jenkins Stage will look like this
```Shell
stage("Pushing Helm Charts to Nexus Repo"){
            steps{
                script{
                    dir('kubernetes/'){
                        withCredentials([string(credentialsId: 'nexus_docker_repo_pass', variable: 'nexus_docker_repo_pass_var')]) {
                                sh '''
                                    helmversion=$(helm show chart myapp/ | grep version | awk '{print $2}')
                                    tar -czvf myapp-${helmversion}.tgz myapp/
                                    curl -u admin:$nexus_docker_repo_pass_var http://52.66.172.21:8081/repository/helm-hosted/ --upload-file myapp-${helmversion}.tgz -v
                                '''
                            }   
                        }
                }
            }
        }
```

4. Now we can see the helm charts in our nexus repo
	![[Pasted image 20230509201746.png]]

5. ![[Pasted image 20230509201847.png]]

	Now whenever we will update our application's helm charts version a new version of the helm charts will be pushed to the nexus repo named `helm-hosted`
	