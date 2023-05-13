
We have already created deployments and services YAML file and helm charts for them which are present in the `kubernetes/myapp` folder in the root directory of the repo.

#### 1. Now, let's add the datree stage for static code analysis of our HELM charts.

```
stage('Identifying the misconfiguration in HELM charts using datree'){
    steps{
        script{
            dir('kubernetes/') {
                withCredentials([string(credentialsId: 'datree-token', variable: 'datree_token_var')]) {
                    sh '''
                    sudo helm datree config set token $datree_token_var
                    sudo helm datree test myapp/
                    '''
                        }    
                    }
                }
            }
        }
```

- First we need to create a secret text credential named `datree-token`
	![[Pasted image 20230511150606.png]]
- Our helm charts are in the kubernetes/myapp directory so the above pipeline does these things.
	- Changes the directory to kubernetes/
	- Sets datree token from the value of jenkins secret credential `datree-token`
	- runs the command helm datree test myapp/

- The Datree token can be accessed by login in to the datree.io dashboard 
	https://app.datree.io/

![[Pasted image 20230509185141.png]]

- All the rules which will be used against our helm charts for code analysis are visible under `Policies` > `Active Rules` 

 ![[Pasted image 20230509185208.png]]

#### 2. Now if we run our build we will see this.

	![[Pasted image 20230509185417.png]]

	![[Pasted image 20230509185451.png]]