
-  We need to do four steps to build our image and push the image to the nexus repository
		- Tag and build docker image
		-  Login to the nexus repo 
		-  Push the docker image to the nexus repo
		-  Remove the image from the server after the image is pushed

#### 1. Tag and build docker image
-  We can tag our image using the command 
	`docker build -t nexus_server_ip:8083/myapp:$VERSION`
- VERSION needs to be unique as every change to the application will trigger a new job and will create a new image so we need a variable which we can use as tag, we can use the build number of our jenkins job as a tag. The image created by the job will have the build number as tag.
- For this to work, first we need to define $VERSION in the pipeline, it's value will be build number of the jenkins job. 
- Define the $VERSION in the pipeline using environment variable which will be avilabe for use during the excution of the pipeline. 
```shell
environment{
        VERSION = "${env.BUILD_ID}"
    }
```
- Now we can use this enviroinment variable in our stage
```Shell
docker build -t 52.66.172.21:8083/myapp:${VERSION} 
```

#### 2. Login to nexus repo
- This can be done using the command 
	`docker login -u admin -p $nexus_pass_var nexus_server_ip:8083`
- `$nexus_pass_var` is variable through which we access the jenkins credential that we created for the nexus repository admin password.
- We will use a withcredentials block to access the credential `nexus_pass`
```Shell
withCredentials([string(credentialsId: 'nexus_pass', variable: 'nexus_pass_var')]) {
	sh '''
	docker build -t nexus_server_ip:8083/myappapp:${VERSION} .
	docker login -u admin -p $nexus_docker_repo_pass_var nexus_server_ip:8083
	//docker push
	//docker rmi
'''
}
```

#### 3. Push the docker image to the nexus repo
- Use this command to push the image to nexus repo
  `docker push nexus_server_ip:8083/myappapp:${VERSION}`

#### 4. Remove the image from the server after the image is pushed
- This can be done using the command
  `docker rmi nexus_server_ip:8083/myapp:${VERSION}`

## Final pipline code for this stage

![[Pasted image 20230511123517.png]]


- Once the build is successfully completed, check the nexus repo docker-hosted to see if the docker images are visible.
  ![[Pasted image 20230511123825.png]]

- Build number is 13, let's check the `docker-hosted` nexus repo if we have the same tag as the build number.
  ![[Pasted image 20230511123936.png]]
  
### Docker image successfully pushed to the nexus repository!!!!!