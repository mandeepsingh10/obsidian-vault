
#### 1. Make sure kubectl is installed on jenkins server

#### 2. Ensure that kubeconfig is present in the home directory of jenkins user
```Shell
jenkins@jenkins:~$ pwd
/var/lib/jenkins ##home dir of jenkins user
jenkins@jenkins:~$ ls -l .kube/
total 12
drwxr-x--- 4 jenkins jenkins 4096 May 11 14:06 cache
-rw-r--r-- 1 root    root    5640 May 11 14:01 config
jenkins@jenkins:~$ 
```

#### 3. Test the k8s-configuration
- Create a test pipeline job to test the connectivity with the k8s0cluster
```Shell
pipeline {
    agent any

    stages {
        stage('K8s Connection Test') {
            steps {
                script{
                    dir ("kubernetes/"){  
				        sh 'kubectl get nodes'
			        }
                }
            }
        }
    }
}
```

- We will see output similar to this if our build is a success and we can connect to the k8s cluster.
	![[Pasted image 20230511210902.png]]


#### 4. Create a secret for nexus docker-hosted repo credentials and connection

`kubectl create secret docker-registry registry-secret --docker-server=15.206.89.243:8083 --docker-username=admin --docker-password=pass1234 --docker-email=not-needed@example.com`

- We need this secret to authenticate to the nexus repo hosted on our nexus server

- This is a kubectl command that creates a Kubernetes secret named `registry-secret` of type `docker-registry`. This secret is used to store credentials for authenticating with a Docker registry located at the specified IP address and port number (`206.89.243:8083`). The `--docker-username`, `--docker-password`, and `--docker-email` flags are used to set the corresponding authentication credentials for the Docker registry.


#### 5. Add the nexus repo on both k8s-master and k8s-node-1

- We need to make both the nodes aware from where they have\to fetch the container images and helm charts.
- **We are using containerd (CRI) so we cannot install docker runtime and configure insecure repo for docker as it will cause many issues in our cluster**
- We will have to configure our nexus repo as an insecure repository for containerd.
- The below mentioned steps need to be performed on both the k8s-nodes (we are only scheduling our deployments on k8s-node1 but it's a good practice to keep the configuration consisitent so we will do it for both the nodes)
   
1. Set the default config directory for container.d
```Shell
mkdir -p /etc/containerd
containerd config default>/etc/containerd/config.toml
systemctl restart containerd
systemctl status  containerd.service 
systemctl enable containerd
vim /etc/containerd/config.toml
```

2. Find line `[plugins."io.containerd.grpc.v1.cri".registry.configs]`

3. Add these six line below it
```Shell
[plugins."io.containerd.grpc.v1.cri".registry.configs."15.206.89.243:8083"]

        [plugins."io.containerd.grpc.v1.cri".registry.configs."15.206.89.243:8083".tls]
                ca_file = ""

                cert_file = ""

                insecure_skip_verify = true

                key_file = ""

```

4. Find the line `[plugins."io.containerd.grpc.v1.cri".registry.mirrors]`
	Add these two lines below it
```Shell
[plugins."io.containerd.grpc.v1.cri".registry.mirrors."15.206.89.243:8083"]
                endpoint = ["http://15.206.89.243:8083"]
```

4. This is the what it will look like after making the changes
```Shell
[plugins."io.containerd.grpc.v1.cri".registry.configs]

    [plugins."io.containerd.grpc.v1.cri".registry.configs."15.206.89.243:8083"]

        [plugins."io.containerd.grpc.v1.cri".registry.configs."15.206.89.243:8083".tls]
			    ca_file = ""

                cert_file = ""

                insecure_skip_verify = true

                key_file = ""
      [plugins."io.containerd.grpc.v1.cri".registry.headers]

      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
         [plugins."io.containerd.grpc.v1.cri".registry.mirrors."15.206.89.243:8083"]
                endpoint = ["http://15.206.89.243:8083"]

```


#### 6. Restart containerd service and try pulling an image from our repo
```Shell
systemctl restart containerd.service
root@k8s-master:~# crictl pull 15.206.89.243:8083/springapp:34
Image is up to date for sha256:2197dad469e378ff389a48f5d12fbd9922503e8751897f31c84446acfe84ff44
root@k8s-master:~# crictl images
IMAGE                                     TAG                 IMAGE ID            SIZE
15.206.89.243:8083/springapp              34                  2197dad469e37       287MB
```

- Now we can pull images from our nexus repo from our k8s nodes

