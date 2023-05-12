1. Find line `[plugins."io.containerd.grpc.v1.cri".registry.configs]`
2. Add these six line below it
```Shell
[plugins."io.containerd.grpc.v1.cri".registry.configs."15.206.89.243:8083"]

        [plugins."io.containerd.grpc.v1.cri".registry.configs."15.206.89.243:8083".tls]
                ca_file = ""

                cert_file = ""

                insecure_skip_verify = true

                key_file = ""

```

3. Find the line `[plugins."io.containerd.grpc.v1.cri".registry.mirrors]`
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