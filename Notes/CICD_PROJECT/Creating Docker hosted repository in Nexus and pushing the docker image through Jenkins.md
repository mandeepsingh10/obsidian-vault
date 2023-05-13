
1.  Create a repository in nexus for Docker images
	- Goto Nexus repo dashboard  > `Repositories` > `Create Repository`
			Select Recipe as `docker-hosted`
			Select HTTP port as `8083`
			Click on `Create repository`
	![[Pasted image 20230511102520.png]]

2. Now we need to configure this repository on our jenkins server as an insecure registry to so that we can push our docker images to this repo.
	 - To do so, go to the jenkins server and create or edit the file `/etc/docker/daemon.json`
```shell
vim /etc/docker/daemon.json
{ "insecure-registries":["nexus_machine_ip:8083"] }
```
- It will look like this `{ "insecure-registries":["15.206.89.243:8083"] }`
- Now restart the docker service using `systemctl restart docker.service`

3. Check if the registry is added properly, run `docker info | grep Insecure -A2`
```Shell
root@jenkins:~# docker info | grep Insecure -A2
 Insecure Registries:
  15.206.89.243:8083
  127.0.0.0/8
```

4. We have successfully created and configured the nexus repository for storing our docker images.
5. Now, let's try to login to the docker repository 
```Shell
root@jenkins:~# docker login -u admin 15.206.89.243:8083
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

6. Now we need to add the password for our nexus repository as a secret in jenkins so that we can use it in our pipeline stages.
    Goto `Manage Jenkins` > `Security` > `Credentials`. 
    Click on `global` and then `add credentials` , select kind as `secret text`
    ![[Pasted image 20230511104419.png]]