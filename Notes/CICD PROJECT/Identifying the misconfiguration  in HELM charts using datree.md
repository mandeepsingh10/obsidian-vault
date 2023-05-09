
We have already created deployments and services YAML file and helm charts for them which are present in the `kubernetes/myapp` folder in the root directory of the repo.

1. Now, let's add the datree stage for static code analysisi of our HELM charts.

```
stage('Identifying the misconfiguration in HELM charts using datree'){

steps{

script{

dir('kubernetes/') {

withEnv(['DATREE_TOKEN=ff434798-2f4e-4adf-a167-41b43740c677']) {

sh 'sudo helm datree test myapp/'

}

}

}

}

}
```

Our helm charts are in the kubernetes/myapp directory so the above pipeline does these things.
	- Changes the directory to kubernetes/
	- Sets an environment variable DATREE_TOKEN
	- runs the command helm datree test myapp/

The Datree token can be accessed by login in to the datree.io dashboard 
https://app.datree.io/

![[Pasted image 20230509185141.png]]

All the rules which will be used against our helm charts for code analysis.

![[Pasted image 20230509185208.png]]

2. Now if we run our build we will see this.

	![[Pasted image 20230509185417.png]]

	![[Pasted image 20230509185451.png]]