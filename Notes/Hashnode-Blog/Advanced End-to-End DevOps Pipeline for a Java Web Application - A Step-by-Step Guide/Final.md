## Overview

This project aims to build an advanced end-to-end DevOps pipeline for a Java web application.  
Our project is divided into two main parts:

1. The initial phase involves the installation and configuration of various tools and servers.
    
2. In the second phase, we will create an advanced end-to-end Jenkins pipeline with multiple stages.
    

## Tools used

1. AWS
    
2. Git
    
3. Gradle
    
4. Ansible
    
5. Terraform
    
6. Jenkins
    
7. SonarQube
    
8. SonaType Nexus Repository Manager
    
9. Kubernetes (kubeadm k8s cluster)
    
10. Helm
    
11. datree.io
    
12. Slack
    

## Prerequisites

1. AWS account
    
2. Terraform and Ansible should be installed and configured on your local computer
    

## **Phase I: Installation and configuration**

* We have used terraform config files to provision AWS resources and ansible playbooks to configure the instances as per our requirement, so you should know how to write terraform config files and ansible playbooks for understanding this phase of the project clearly. If you want to test the project then you can just clone the repo and run the commands which will automatically set up the required AWS resources.
    
* We need five servers to set up our CICD pipeline infrastructure. We have used **Ubuntu** as the base operating system for all our servers.
    
    1. Jenkins Server - `jenkins`
        
        * This will host our jenkins application to implement our CI/CD pipelines.
            
    2. SonarQube Server - `sonar`
        
        * This server will host the SonarQube application. It is used for continuous inspection of code quality to perform automatic reviews with static analysis of code to detect bugs and code smells.
            
    3. Nexus Repository server - `nexus`
        
        * Sonatype **Nexus Repository** Manager provides a central platform for storing build artifacts. This server will host our two private repositories one for storing docker images and another for storing helm charts.
            
    4. Kubernetes control plane node - `k8s-master`
        
        * This server will act as the control plane node for our kubernetes cluster.
            
    5. Kubernetes worker node - `k8s-node1`
        
        * The `k8s-node1` server will act as a worker node for our kubernetes cluster
            
* This concludes the brief description of the servers needed for our project. We can now proceed to **Phase I**.
    

### Setup AWS account for Terraform

1. To be able to set up the AWS resources with Terraform, we need to access AWS using **Access Keys**
    
2. Create a new user with administrator access to the AWS account
    
    * Goto **IAM &gt; Users**, click on **Add users** and enter the user name and click **Next**.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683891310377/713eeff5-b1c2-4aee-ac2f-1e53827e76e2.png align="center")
        
    * On the next page, select **Attach policies directly** and attach the **AdministratorAccess** policy to the user. Click on **Next**, then click on **Create user.**
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683891404598/c37896dd-1412-4ee1-b697-73f7032bcae4.png align="center")
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683891491991/6d48bb61-6804-4064-9a9b-6492eeaa0e8e.png align="center")
        
3. Create an AWS access key for the newly created user.
    
    * Navigate to **IAM &gt; Users &gt; tf-user**, select **Security credentials** and click on **Create access key**, select **Other** and click on **Next**.
        
    * Enter a name for your access key and click on **Create access key** to create the key.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683892022225/bd64e2f0-2655-43bd-a08b-34e866505f12.png align="center")
        
    * Next, you will get an option to download the `.csv` file containing the access key and secret key or you can simply copy and store them in a safe place
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683892196190/40d5ebb7-e492-4ec2-8430-3bba8c6e0765.png align="center")
        
4. Set environment variables on your local system for Terraform to access the AWS access key.
    
    * Create a file called `.envrc` in your home directory and write the commands to export the environment variables in it.
        
        ```bash
        #!/bin/bash
        
        ##Access_keys for AWS Terraform User##
        export TF_VAR_access_key="Enter the aws access key here"
        export TF_VAR_secret_key="Enter the secret access key here"
        ```
        
        Replace the data with your AWS access and secret keys and save the file. We have used the same variables in our terraform config files so make sure the name of the environment variables are exactly as mentioned in the code block above.
        
    * Edit your `.bashrc` file and add this line at the end `source ~/.envrc`  
        save the file and exit.
        

### Clone the Git repository

```bash
git clone https://github.com/mandeepsingh10/cicd-setup.git
```

1. In the git repo, we have three directories.
    
    * **ansible\_config** - contains all the ansible playbooks
        
    * **terraform\_config** - contains all the terraform config files
        
    * **scripts** - contains miscellaneous scripts to automate some tasks
        
2. To provision AWS instances using Terraform, we need to specify a **public key** to access the resources, we can use our public key for this instead of creating a new pair as it will help us to directly access the `EC2` instances without doing any additional configuration for ssh.
    
3. Copy your **public key** from `~/.ssh` directory to every Terraform configuration directory under `terraform_config` directory in the repository.
    
    ```bash
    cp $HOME/.ssh/id_rsa.pub $HOME/repos/cicd-setup/terraform_config/jenkins/publickey.pub
    ```
    
    My repository is present at `$HOME/repos` path, change that path as per your repo path and copy the public key as `publickey.pub` to all the directories `jenkins`, `master`, `nexus`, `node1` and `sonar` under `terraform_config` directory.
    
4. Now we will be able to provision the terraform resources by just running the `terraform apply` command.
    

### Create AWS instances using Terraform

1. First, we need to initialize all the directories where we have our Terraform configuration files.
    
    ```bash
    cd  $HOME/repos/cicd-setup/terraform_config/jenkins
    ➜ terraform init
    ```
    
    We need to initialize all the directories where we have our Terraform configuration files. To do this, navigate to each directory under `cicd/terraform_config` - `jenkins`, `master`, `nexus`, `node1`, and `sonar` - and run the command `terraform init`.
    
2. Next, we can try provisioning any one of the instances to check if our configuration is working as expected. To do this, navigate to the `Jenkins` directory and run the command `terraform plan`. If everything is set up correctly, it should display a message similar to this at the end:
    
    ```bash
    Changes to Outputs:
      + Name                 = "jenkins"
      + ami_id               = "ami-0bcb40eb5cb6d6f9e"
      + instance_id          = (known after apply)
      + keyname              = "jenkins-key"
      + public_dns           = (known after apply)
      + public_ip            = (known after apply)
      + securityGroupDetails = (known after apply)
    ```
    
    This means that our terraform configuration is correct and is working as expected.
    
3. To provision the jenkins server on AWS by running the command  
    `terraform apply --auto-approve` in the `cicd-setup/terraform_config/jenkins` directory.
    
4. Once the server is provisioned, the command will display details about the server, such as the hostname, `ami_id` of the image used, public and private IP addresses, similar to this:
    
    ```bash
    Apply complete! Resources: 3 added, 0 changed, 0 destroyed.
    
    Outputs:
    
    Name = "jenkins"
    ami_id = "ami-0bcb40eb5cb6d6f9e"
    instance_id = "i-09a9d9b364b692fb7"
    keyname = "jenkins-key"
    public_dns = "ec2-3-109-154-107.ap-south-1.compute.amazonaws.com"
    public_ip = "3.109.154.107"
    securityGroupDetails = "sg-02b24f714c8b6b86a"
    ```
    
5. Let's try accessing the `jenkins` server by `ssh` using it's public IP. By default, `ubuntu` user is created on all our servers, as we are using **Ubuntu** ami image.
    
    ```bash
    ssh ubuntu@3.109.154.107
    ubuntu@jenkins:~$
    ```
    
    We can access the Jenkins server, which means we are good to go.
    
6. We can automate the provisioning of all the AWS resources using the `t_createall.sh` script located in the `cicd-setup/scripts`. While implementing Phase I for the first time, I recommend against using the `t_createall.sh` script to create all necessary resources at once. This is because our infrastructure setup utilizes `t2.medium` EC2 instances, and creating all resources simultaneously may result in higher AWS EC2 costs if instances remain idle while we configure other servers. Additionally, running a script may not provide a thorough understanding of how things work.
    
7. Let's move on to our next step, which is configuring our servers using Ansible playbooks. We can refer to this section whenever we need to provision the remaining servers on AWS.
    

### Configure the servers using Ansible

I have created Ansible playbooks to configure all the servers according to the application/service we need to run on them.  
The `playall.sh` script provides an option to automate the configuration management of all servers. However, as previously mentioned, solely relying on a script may not provide a comprehensive understanding of the underlying processes involved.

1. **Jenkins server**
    
    * Let's start with the `jenkins` server, but first, we need to populate our Ansible inventory file with the public IP addresses of our AWS instances so that we can run our Ansible playbooks. To do so, run the `generate_ansible_inventory.sh` from the `cicd-setup/scripts` directory.
        
        ```bash
        ~/repo/cicd-setup/scripts$
        ➜ ./generate_ansible_inventory.sh
        
        Creating inventory file for ansible ....
        
        Populating inventory file for ansible ....
        
        Done!!!
        ```
        
        Check the newly created inventory file under `ansible_config` directory.
        
        ```bash
        ~/repo/cicd-setup$
        ➜ cat ansible_config/inventory
        jenkins ansible_host=3.109.154.107 ansible_user=ubuntu ansible_connection=ssh
        sonar ansible_host= ansible_user=ubuntu ansible_connection=ssh
        nexus ansible_host= ansible_user=ubuntu ansible_connection=ssh
        k8s-master ansible_host= ansible_user=ubuntu ansible_connection=ssh
        k8s-node1 ansible_host= ansible_user=ubuntu ansible_connection=ssh
        ```
        
        We can see that the file is generated with the details required by Ansible to connect to the `jenkins` server. As we haven't provisioned the remaining servers yet, the IP details are empty for them.
        
    * Now, to configure the `jenkins` server as per our requirements for this project, simply run the Ansible playbook located at `ansible_config/jenkins/jenkins.yaml`. You can check the Ansible playbook to understand how we are configuring the `jenkins` server. The playbook output shows us what all tasks are being performed.
        
        ```yaml
        ~/repos/cicd-setup/ansible_config$ 
        ➜ ansible-playbook -i inventory jenkins/jenkins.yaml
        
        PLAY [Install and start Jenkins] ********************************************************************************************************************************************
        
        TASK [Gathering Facts] ******************************************************************************************************************************************************
        ok: [jenkins]
        
        TASK [ensure the jenkins apt repository key is installed] *******************************************************************************************************************
        changed: [jenkins]
        
        TASK [ensure the repository is configured] **********************************************************************************************************************************
        changed: [jenkins]
        
        TASK [Ensure java is installed] *********************************************************************************************************************************************
        changed: [jenkins]
        
        TASK [ensure jenkins is installed] ******************************************************************************************************************************************
        changed: [jenkins]
        
        TASK [ensure jenkins is running] ********************************************************************************************************************************************
        ok: [jenkins]
        
        PLAY [Install Helm and datree.io helm plugin] *******************************************************************************************************************************
        
        TASK [Gathering Facts] ******************************************************************************************************************************************************
        ok: [jenkins]
        
        TASK [Check if Helm is installed] *******************************************************************************************************************************************
        fatal: [jenkins]: FAILED! => {"changed": false, "cmd": "helm version", "delta": "0:00:00.002648", "end": "2023-05-12 18:31:45.516957", "msg": "non-zero return code", "rc": 127, "start": "2023-05-12 18:31:45.514309", "stderr": "/bin/sh: 1: helm: not found", "stderr_lines": ["/bin/sh: 1: helm: not found"], "stdout": "", "stdout_lines": []}
        ...ignoring
        
        TASK [Download Helm script] *************************************************************************************************************************************************
        changed: [jenkins]
        
        TASK [Install Helm] *********************************************************************************************************************************************************
        changed: [jenkins]
        
        TASK [Install unzip] ********************************************************************************************************************************************************
        changed: [jenkins]
        
        TASK [Check if Datree plugin exists] ****************************************************************************************************************************************
        ok: [jenkins]
        
        TASK [Install Datree plugin] ************************************************************************************************************************************************
        changed: [jenkins]
        
        PLAY [Install Docker] *******************************************************************************************************************************************************
        
        TASK [Gathering Facts] ******************************************************************************************************************************************************
        ok: [jenkins]
        
        TASK [Install required packages] ********************************************************************************************************************************************
        changed: [jenkins]
        
        TASK [Add Docker GPG key] ***************************************************************************************************************************************************
        changed: [jenkins]
        
        TASK [Add Docker repository] ************************************************************************************************************************************************
        changed: [jenkins]
        
        TASK [Install Docker] *******************************************************************************************************************************************************
        changed: [jenkins]
        
        TASK [Add jenkins to sudo group] ********************************************************************************************************************************************
        changed: [jenkins]
        
        TASK [Add jenkins to docker group] ******************************************************************************************************************************************
        changed: [jenkins]
        
        TASK [Add jenkins to sudoers file] ******************************************************************************************************************************************
        changed: [jenkins]
        
        TASK [Install kubectl] ******************************************************************************************************************************************************
        changed: [jenkins]
        
        TASK [Create kubectl alias for root user] ***********************************************************************************************************************************
        changed: [jenkins]
        
        PLAY RECAP ******************************************************************************************************************************************************************
        jenkins                    : ok=23   changed=17   unreachable=0    failed=0    skipped=0    rescued=0    ignored=1
        ```
        
    * If the playbook is executed without any errors, we should be able to access the Jenkins application from your browser using the public IP address followed by port `8080`.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683926674347/eacbd437-7ac0-40dd-960f-ad0c88c3a95e.png align="center")
        
    * Enter the `initial admin password` and click on `continue`.
        
    * Select `Installed suggested plugins`.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683927643846/4323b18a-1167-4b3a-b49b-d4e8c593f017.png align="center")
        
    * Now jenkins will install the suggested plugins, it will take 5-10 minutes for this to complete.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683927715907/0739a4d9-4458-4e5c-aacd-7c551481e59b.png align="center")
        
    * Once the suggested plugins are installed, you will be prompted to create a user. Create the user according to your preferences.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683927845150/6431505d-3c53-4402-a71e-7e02aabb7316.png align="center")
        
    * Next, you will see this screen, click on `Save and Finish` and then click on `Start using Jenkins` on the next screen.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683927939348/bfd0c9de-f351-4bcb-9cb9-87e5392a45e1.png align="center")
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683928024864/37b44993-3c20-4d84-8a79-7ac348769655.png align="center")
        
    * Finally, you will see the Jenkins dashboard, setup is completed.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683928078410/14f5598c-7a29-4907-853a-716b01e51d00.png align="center")
        
2. **SonarQube Server**
    
    * Follow the steps mentioned in **Create AWS instances using Terraform** section to provision the **SonarQube** server - `sonar`.
        
    * Once the server is provisioned, we need to repopulate the Ansible inventory file before running the playbook to configure sonarqube server.
        
        Run the script `cicd-setup/scripts/generate_ansible_inventory.sh`.
        
    * Run the Ansible playbook located at `ansible_config/sonar/sonar.yaml`.
        
        ```yaml
        ~/repos/cicd-setup/ansible_config$ 
        ➜ ansible-playbook -i inventory sonar/sonar.yaml
        
        PLAY [Install Docker and SonarQube] *****************************************************************************************************************************************
        
        TASK [Gathering Facts] ******************************************************************************************************************************************************
        ok: [sonar]
        
        TASK [Install required packages] ********************************************************************************************************************************************
        changed: [sonar]
        
        TASK [Add Docker GPG key] ***************************************************************************************************************************************************
        changed: [sonar]
        
        TASK [Add Docker repository] ************************************************************************************************************************************************
        changed: [sonar]
        
        TASK [Install Docker] *******************************************************************************************************************************************************
        changed: [sonar]
        
        TASK [Start SonarQube container] ********************************************************************************************************************************************
        changed: [sonar]
        
        PLAY RECAP ******************************************************************************************************************************************************************
        sonar                      : ok=6    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
        ```
        
    * We have deployed our SonarQube application on a docker container and exposed it on `9000` port. We should be able to access the Jenkins application from your browser using the public IP address followed by port `9000`.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683929751575/19fdc15a-8b99-49d4-80e1-9087ad986081.png align="center")
        
    * The default username is `admin` and password is also `admin`.
        
    * Next you will be promted to set a new password for the SonarQube application, set the new password and click on `update`.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683929961354/a98b0901-6bd8-4742-97e6-cef3b30eba41.png align="center")
        
    * Now you will see the SonarQube dashboard, setup is complete.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683930049739/d34807d7-e2a0-4cc0-9f95-1ff6176abf01.png align="center")
        
3. **SonaType Nexus Server**
    
    * Follow the steps mentioned in **Create AWS instances using Terraform** section to provision the **Nexus Repository Manager** server - `nexus`.
        
    * Once the server is provisioned, we need to repopulate the Ansible inventory file before running the playbook to configure nexus server.
        
        Run the script `cicd-setup/scripts/generate_ansible_inventory.sh`.
        
    * Run the Ansible playbook located at `ansible_config/nexus/nexus.yaml`.
        
        ```yaml
        ~/repos/cicd-setup/ansible_config$ 
        ➜ ansible-playbook -i inventory nexus/nexus.yaml
        
        PLAY [Install Nexus] ********************************************************************************************************************************************************
        
        TASK [Gathering Facts] ******************************************************************************************************************************************************
        ok: [nexus]
        
        TASK [Install Java 8] *******************************************************************************************************************************************************
        changed: [nexus]
        
        TASK [Download Nexus] *******************************************************************************************************************************************************
        changed: [nexus]
        
        TASK [Extract Nexus] ********************************************************************************************************************************************************
        changed: [nexus]
        
        TASK [Start Nexus] **********************************************************************************************************************************************************
        changed: [nexus]
        
        PLAY RECAP ******************************************************************************************************************************************************************
        nexus                      : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
        ```
        
    * Nexus takes some time to load so wait for 2-3 minutes and then access it using the public IP address followed by port `8081`.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683931377652/8a48d8c8-89d1-457b-b99a-dc476b61bcdb.png align="center")
        
    * Click on "Sign In" and follow the on-screen instructions to get the initial admin password for Nexus. You will be prompted to set a new password for **Nexus**. Make sure to store this password as we will need it in Phase II of our project where we integrate different applications together.
        
    * On the next screen make sure `Enable anonymous access` is selected and then click on `Next`.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683932180954/ec61b9f8-c258-4083-9ae7-58d907715640.png align="center")
        
    * Click on `Finish`, the setup is completed.
        
4. **Kubernetes Cluster**
    
    * In this section we will setup and configure a kubernetes cluster on AWS using kubeadm tool.
        
    * We will be deploying two k8s nodes, one control plane node `k8s-master` and one worker node `k8s-node1`
        
    * Follow the steps mentioned in **Create AWS instances using Terraform** section to provision the `k8s-master` and `k8s-node1` servers for our kubernetes cluster.
        
    * Once both the servers are provisioned, we need to repopulate the Ansible inventory file before running the playbook to setup and configure our kubernetes cluster.
        
        Run the script `cicd-setup/scripts/generate_ansible_inventory.sh`.
        
    * Run the Ansible playbook located at `ansible_config/k8s/k8s_cluster_setup.yaml`.
        
        ```yaml
        ~/repos/cicd-setup/ansible_config$ 
        ➜ ansible-playbook -i inventory k8s/k8s_cluster_setup.yaml
        
        PLAY [Add private IP of k8s instances to init scripts] **********************************************************************************************************************
        
        TASK [Gathering Facts] ******************************************************************************************************************************************************
        ok: [localhost]
        
        TASK [Run add_k8s_ip.sh] ****************************************************************************************************************************************************
        changed: [localhost]
        
        PLAY [Setup k8s Control Plane (master) node] ********************************************************************************************************************************
        
        TASK [Gathering Facts] ******************************************************************************************************************************************************
        ok: [k8s-master]
        
        TASK [Copy master.sh script to remote server] *******************************************************************************************************************************
        changed: [k8s-master]
        
        TASK [Configure Control Plane (master) node] ********************************************************************************************************************************
        changed: [k8s-master]
        
        TASK [Get kubeadm join command] *********************************************************************************************************************************************
        changed: [k8s-master]
        
        TASK [Generate token to join the k8s cluster] *******************************************************************************************************************************
        changed: [k8s-master]
        
        TASK [Create join_cluster.sh] ***********************************************************************************************************************************************
        changed: [k8s-master -> localhost]
        
        TASK [Create kubectl alias] *************************************************************************************************************************************************
        changed: [k8s-master]
        
        PLAY [Configure kubectl for ubuntu user] ************************************************************************************************************************************
        
        TASK [Gathering Facts] ******************************************************************************************************************************************************
        ok: [k8s-master]
        
        TASK [Create .kube directory for ubuntu user] *******************************************************************************************************************************
        changed: [k8s-master]
        
        TASK [Copy admin.conf to ubuntu's .kube directory] **************************************************************************************************************************
        changed: [k8s-master]
        
        TASK [Change ownership of .kube/config file] ********************************************************************************************************************************
        ok: [k8s-master]
        
        TASK [Create kubectl alias for ubuntu] **************************************************************************************************************************************
        changed: [k8s-master]
        
        PLAY [Setup k8s worker node] ************************************************************************************************************************************************
        
        TASK [Gathering Facts] ******************************************************************************************************************************************************
        ok: [k8s-node1]
        
        TASK [Copy nodes.sh script to remote server] ********************************************************************************************************************************
        changed: [k8s-node1]
        
        TASK [Configure Worker node] ************************************************************************************************************************************************
        changed: [k8s-node1]
        
        TASK [Copy join_cluster.sh] *************************************************************************************************************************************************
        changed: [k8s-node1]
        
        TASK [Join k8s cluster] *****************************************************************************************************************************************************
        changed: [k8s-node1]
        
        PLAY [Copy the kubeconfig form k8s-master] **********************************************************************************************************************************
        
        TASK [Gathering Facts] ******************************************************************************************************************************************************
        ok: [k8s-master]
        
        TASK [Fetch the file from the k8s-master to localhost] **********************************************************************************************************************
        changed: [k8s-master]
        
        PLAY [Jenkins User kubectl Setup] *******************************************************************************************************************************************
        
        TASK [Gathering Facts] ******************************************************************************************************************************************************
        ok: [jenkins]
        
        TASK [create directory and set ownership of .kube directory for jenkins user] ***********************************************************************************************
        changed: [jenkins]
        
        TASK [Copy the file from localhost to jenkins] ******************************************************************************************************************************
        changed: [jenkins]
        
        PLAY RECAP ******************************************************************************************************************************************************************
        jenkins                    : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
        k8s-master                 : ok=14   changed=10   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
        k8s-node1                  : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
        localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
        ```
        
    * Once the playbook is successfully executed, we can login to our master nodes to verify the setup and configuration of our k8s cluster.
        
        ```bash
        ssh ubuntu@65.0.21.196
        
        ubuntu@k8s-master:~$ kubectl get nodes
        NAME         STATUS   ROLES           AGE     VERSION
        k8s-master   Ready    control-plane   11m     v1.27.1
        k8s-node1    Ready    <none>          9m39s   v1.27.1
        
        ubuntu@k8s-master:~$ kubectl get pods -n kube-system
        NAME                                 READY   STATUS    RESTARTS      AGE
        coredns-5d78c9869d-blhx8             1/1     Running   0             21m
        coredns-5d78c9869d-tfzj6             1/1     Running   0             21m
        etcd-k8s-master                      1/1     Running   0             21m
        kube-apiserver-k8s-master            1/1     Running   0             21m
        kube-controller-manager-k8s-master   1/1     Running   0             21m
        kube-proxy-qsp48                     1/1     Running   0             19m
        kube-proxy-rv87l                     1/1     Running   0             21m
        kube-scheduler-k8s-master            1/1     Running   0             21m
        weave-net-mx4lj                      2/2     Running   0             19m
        weave-net-zddmr                      2/2     Running   1 (20m ago)   21m
        ```
        
    * We can see that both nodes are in a Ready state, and all pods in the kube-system namespace are up and running, indicating that our k8s cluster has been deployed properly.
        
    
    #### With this we have successfully completed Phase I of our project, the required infrastructure for the CICD pipeline is successfully provisioned and configured.
    
    **Note:**  
    <mark>When implementing Phase I for the first time, it may take a considerable amount of time to complete. If you need to take a break or are unable to continue for a few days, it is not advisable to keep the AWS resources running during this time as it can result in significantly higher charges.</mark>
    
    For such scenarios or if you make mistakes during Phase I and need to start over but don't want to repeat the same steps again, you can use the following scripts in the cicd-setup/scripts directory:
    
    1. `t_createall.sh`: provisions all EC2 instances
        
    2. `t_destroyall.sh`: destroys all EC2 instances
        
    3. `playall.sh`: automates the configuration management of all servers
        
    4. `phase1.sh`: automates both provisioning and configuration management processes
        
    
    After automating the provisioning and configuration management processes, you will still need to complete certain manual steps in the initial setup of applications such as Jenkins, SonarQube, and Nexus. To do this, you can access the dashboards of each application using the public IP followed by the port in your web browser.
    
    That's it for Phase I, let's proceed to Phase II.
    

## **Phase II: The CICD Pipeline**

* In this phase we will do everything from integrating our Jenkins, SonarQube, Nexus Repository Manager, Kubernetes cluster, Helm, datree.io applications with each other to deploying our Java web application on the K8s cluster using Helm charts based on pull request triggers. Let's begin.
    

### **Clone the git repository**

* We will be updating and pushing the changes to the repository so we need to clone the repository via ssh to do so first create a fork of the repository and then
    
    clone it using the ssh URL.
    
    ```bash
    git clone git@github.com:mandeepsingh10/java-gradle-app.git
    ```
    
* Currently we only have `main` branch in the repository, so we will create a new branch called `dev` as it is not a good practice to modify the `main` branch.
    
    ```bash
    java-gradle-app on main via 🅶 v7.1.1 via ☕ 
    ➜ git branch
    * main
    java-gradle-app on main via 🅶 v7.1.1 via ☕ 
    ➜ git checkout -b dev
    Switched to a new branch 'dev'
    java-gradle-app on dev via 🅶 v7.1.1 via ☕ 
    ➜ git branch
    * dev
      main
    ```
    

### Creating a pipeline job

* Go to the jenkins dashboard and click on `New Item`, select the type as `Pipeline` and Enter a name for the pipeline. Click on `OK`.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683972152743/08d2ed68-4971-45f1-b20d-18fe002d4645.png align="center")
    
* Navigate to the `Pipeline` section of the job and select `Pipeline script from SCM`, select `SCM` as `Git`, enter the repository URL, leave the `Credentials` field empty as our repo is public. In the `Branches to build` section, select `Branch Specifier` as the newly created `dev` branch. Click on `Save`.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683972707506/c0ad3563-7090-444a-9576-b21c42878109.png align="center")
    
* Now, let's create a `Jenkinsfile` in the dev branch with a simple one stage pipeline to check if our code is getting pulled properly or not. Open your favorite code editor and create the `Jenkinsfile` in the root directory of the repo, make sure you are checked out to `dev` branch.
    
    ```bash
    pipeline{
        agent any
        stages{
            stage("Git Test"){
                steps{
                    echo "Executing git connection test"
                }
                post{
                    success{
                        echo "git repository cloned successfully"
                    }
                    failure{
                        echo "git clone action failed"
                    }
                }
            }
        }
        post{
            success{
                echo "========pipeline executed successfully ========"
            }
            failure{
                echo "========pipeline execution failed========"
            }
        }
    }
    ```
    
* Add the Jenkinsfile to dev branch and commit the changes.
    
    ```bash
    java-gradle-app on  dev  🛤️ ×1via 🅶 v7.1.1 via ☕ 
    ➜ git branch
    * dev
      main
    
    java-gradle-app on  dev  🛤️ ×1via 🅶 v7.1.1 via ☕ 
    ➜ git status
    On branch dev
    Untracked files:
      (use "git add <file>..." to include in what will be committed)
    	Jenkinsfile
    
    nothing added to commit but untracked files present (use "git add" to track)
    
    java-gradle-app on  dev  🛤️ ×1via 🅶 v7.1.1 via ☕ 
    ➜ git add .
    
    java-gradle-app on  dev  🗃️×1via 🅶 v7.1.1 via ☕ 
    ➜ git commit -m "Added Jenkinsfile"
    [dev 5b6b195] Added Jenkinsfile
     1 file changed, 26 insertions(+)
     create mode 100644 Jenkinsfile
    
    java-gradle-app on  dev via 🅶 v7.1.1 via ☕ 
    ➜ git push
    fatal: The current branch dev has no upstream branch.
    To push the current branch and set the remote as upstream, use
    
        git push --set-upstream origin dev
    
    java-gradle-app on  dev via 🅶 v7.1.1 via ☕ 
    ✖  git push --set-upstream origin dev
    Enumerating objects: 4, done.
    Counting objects: 100% (4/4), done.
    Delta compression using up to 8 threads
    Compressing objects: 100% (3/3), done.
    Writing objects: 100% (3/3), 459 bytes | 459.00 KiB/s, done.
    Total 3 (delta 1), reused 0 (delta 0), pack-reused 0
    remote: Resolving deltas: 100% (1/1), completed with 1 local object.
    remote: 
    remote: Create a pull request for 'dev' on GitHub by visiting:
    remote:      https://github.com/mandeepsingh10/java-gradle-app/pull/new/dev
    remote: 
    To github.com:mandeepsingh10/java-gradle-app.git
     * [new branch]      dev -> dev
    Branch 'dev' set up to track remote branch 'dev' from 'origin'.
    ```
    
    We received an error saying `The current branch dev has no upstream branch`. This was because we created a new branch but didn't set the remote upstream for it.  
    We ran the command `git push --set-upstream origin dev` to push the changes after setting the upstream.
    
* Now, let's try building our pipeline job in jenkins. Goto Jenkins dashboard and then select `java-gradle-app` and click on `Build Now`.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683974174403/e6dd8d48-754e-46f1-b0aa-a6786ae20550.png align="center")
    
* Our test build was successfull. Let's check the logs, click on the `#1` build and then select `Console Output`.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683974333818/e7cf2685-0aec-4cdf-bb7c-ceb6d93a2f09.png align="center")
    
    We can see the commands that were executed during the execution and some other details, this completes our test run, now let's integrate SonarQube with our Jenkins server.
    

### **STAGE I : SonarQube Integration**

* In this step, we will integrate our SonarQube server with our Jenkins server so that we can perform Code Quality checks, code smells etc.
    
* To start, we'll need to install some jenkins plugins, to do so, navigate to  
    `Manage Jenkins` &gt; `Manage Plugins` &gt; `Available` and install the plugins listed below:
    
    1. SonarQube Scanner for Jenkins
        
    2. Sonar Gerrit
        
    3. Sonar Quality Gates
        
    4. SonarQube Generic Coverage
        
    5. Quality Gates
        
    6. Docker
        
    7. Docker Pipeline
        
    8. docker-build-step
        
    
    Click on `Install without restart`, it will take some time to install the plugins.
    
* Next, we will establish connection between SonarQube and Jenkins Server for that we will need an authentication token from SonarQube.  
    Goto SonarQube dashboard, goto `Administration` &gt; `Security`, generate a token for administrator user by clicking on the icon below the Tokens column.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683975468693/f2d0ec24-9096-450b-8f31-53e56b989abd.png align="center")
    
    Choose name as jenkins and leave the `Expires in` field at the default value of 30 days and click on `Generate`.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683975624110/68cd252e-aa48-4c68-8cb6-6c55df8a9c11.png align="center")
    
* Copy this token, we will create a Jenkins secret using this token which will used to authenticate jenkins while connecting with sonarqube.  
    Goto `Manage Jenkins` &gt; `Security` &gt; `Credentials`. Click on `global` and then `add credentials`, select `Kind` as `Secret text` and `ID` as `sonar-token`
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683976554709/e9c41709-893f-40b1-8e56-6ee73f54a9f5.png align="center")
    
* Now, we need to configure SonarQube settings, Goto `Manage Jenkins` &gt; `Configure System` . There will be a section `SonarQube servers`, we need to update the details in this section. Click on `Add SonarQube`, Enter the name as `sonarserver`, Server URL is `http://IP:PORT`, select `Server authentication token` as `sonar-token`. Also enable `Environment variables`.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683980423832/9f919d44-7682-4a42-a64b-efd50918736b.png align="center")
    
* We need to use this token in our Jenkinsfile so we will need to generate a pipeline script for that, navigate to the pipeline job and click on `Pipeline Syntax` and generate sonarqube pipeline script from the **Snippet Generator**.  
    Select `Sample Step` as `withSonarQubeEnv` , select `Server authentication token` as `sonar-token`, click on `Generate Pipeline Script`.
    
    ```bash
    withSonarQubeEnv {
        // some block
    }
    ```
    
    This is the pipeline script generated, similarly whenever we need to generate any script we can use the **Snipper Generator**.
    
* Now add this pipeline script to the Jenkinsfile. Remove the previous test stages and create a new Stage `Sonar Quality Check` where we will perform code analysis by querying SonarQube.
    
    ```bash
    pipeline{
        agent any
        environment{
            VERSION = "{env.BUILD_ID}"
        }
        stages{
            stage("Sonar Quality Check"){
                steps{
                    script{
                        withSonarQubeEnv(credentialsId: 'sonar-token') {
                            sh 'chmod +x gradlew'
                            sh './gradlew sonarqube --info'
                        }
                    }
                }
            }
        }
    }
    ```
    
* We have give execute permission to gradlew and then we have executed `./gradlew sonarqube` which helps us in pushing the code to sonarQube, where we will validate our checks against the sonar rules.
    
* Add the JenkinsFile to the git repo and push the changes.
    
* Go to the Jenkins dashboard and build the pipeline job and see if it builds successfully.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683978469313/01e9d425-e91e-420d-80ac-937e37cf07dc.png align="center")
    
* We can see that the job was a success, we can also check the SonarQube Dashboard.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683987007802/3c8672b4-7290-453c-88b1-85bf485a2fcd.png align="center")
    
* Now, we will we also need to create a webhook to have connection between jenkins and sonarqube.
    
    1. To create a webhook navigate to sonarqube dashboard then `administration > configuration > webhooks`.
        
    2. Enter the URL of your jenkins host with port suffixed with  
        `/sonarqube-webhook/` - The URL will be [`http://jenkins_ip:8080/sonarqube-webhook/`](http://jenkins_ip:8080/sonarqube-webhook/)
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683978754968/5d85ed7a-3bdf-4422-89fe-2dd9bab7c7cc.png align="center")
        
* The data that will be sent to jenkins via the sonarqube webhook can be seen by clicking the small icon beside the time in the `Last delivery` column (it will be available after we start a new build after creating the webhook).  
    Go back to jenkins dashboard and click on `Build Now`. Once the build is completed successfully, go back to SonarQube Dashboard and check the Last Delivery Data.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683979006756/4e6ce4b4-ee17-4ba1-a1fb-ec32aae374f6.png align="center")
    
* We are interested in the qualityGate section of the `Last Delivery` data.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683981545367/c6ed3a79-d765-4f2b-8de3-0d63d86f50ff.png align="center")
    
    ```bash
    "qualityGate": {
    		    "name": "Sonar way",
    		    "status": "OK",
    ```
    
* If the status is `ok` then only our pipeline will move on to the next stage otherwise the build will fail.
    
* Now, we will add a block to our `Sonar Quality Check` stage which will wait for 15 minutes for the Quality Gate status to chaneg to `ok` state.
    
    ```bash
    timeout(time: 15, unit: 'MINUTES') {
        def qg = waitForQualityGate()
        if (qg.status != 'OK') {
            error "Pipeline aborted due to quality gate failure: ${qg.status}"
            }
        }
    ```
    
    ```bash
    pipeline{
        agent any
        environment{
            VERSION = "{env.BUILD_ID}"
        }
        stages{
            stage("Sonar Quality Check"){
                steps{
                    script{
                        withSonarQubeEnv(credentialsId: 'sonar-token') {
                            sh 'chmod +x gradlew'
                            sh './gradlew sonarqube --info'
                        }
                        timeout(time: 15, unit: 'MINUTES') {
                            def qg = waitForQualityGate()
                            if (qg.status != 'OK') {
                                error "Pipeline aborted due to quality gate failure: ${qg.status}"
                            }
                        }
                    }
                }
            }
        }
    }
    ```
    
* Commit and push the changes to the `dev` branch and then run the build.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683986945933/34ecc0b0-4cb9-4611-a1b7-876d134975aa.png align="center")
    
* This concludes the SonarQube Integration.
    

### STAGE II : Build docker images and push to Nexus

1. **Create a private repository in Nexus Repository Manager**
    
    * We will create a private repository to store our docker images on the Nexus Repository Manager.
        
        * Go to Nexus dashboard &gt; `Repositories` &gt; `Create Repository` Select Recipe as `docker-hosted` Select HTTP port as `8083` Click on `Create repository`
            
            ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683982174104/25f7156c-fd7b-4aa5-86e7-31dec897e891.png align="center")
            
        * Next, we need to configure this repository on our jenkins server as an insecure registry to so that we can push our docker images to this repo.  
            To do so, go to the jenkins server and create or edit the file `/etc/docker/daemon.json`
            
            ```bash
            vim /etc/docker/daemon.json
            { "insecure-registries":["nexus_machine_ip:8083"] }
            ```
            
            It will look like this `{ "insecure-registries":["13.235.91.151:8083"] }`
            
        * Now restart the docker service using `systemctl restart docker.service` and check if the insecure registry is added properly.
            
            ```bash
            root@jenkins:~# docker info | grep Insecure -A1
             Insecure Registries:
              13.235.91.151:8083
            ```
            
        * We have successfully created and configured the nexus repository for storing our docker images, now let's try to login to the repository.
            
            ```bash
            root@jenkins:~# docker login -u admin 13.235.91.151:8083
            Password: 
            WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
            Configure a credential helper to remove this warning. See
            https://docs.docker.com/engine/reference/commandline/login/#credentials-store
            Login Succeeded
            ```
            
        * Next, we need to add the password for our nexus repository as a secret in jenkins so that we can use it in our pipeline stages.  
            Goto `Manage Jenkins` &gt; `Security` &gt; `Credentials`. Click on `global` and then `add credentials` , select kind as `secret text`. Click on `Create`.
            
            ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683982942553/4cfbb21f-180b-4217-8d7b-83eef7bd7648.png align="center")
            
2. **Creating a multi-stage Dockerfile**
    
    * This Dockerfile is creating a Docker image that deploys a Java web application to Tomcat using Docker multi-stage build.
        
        ```bash
        FROM openjdk:11 as base
        
        WORKDIR /app
        
        COPY . .
        
        RUN chmod +x gradlew
        
        RUN ./gradlew build
        
        FROM tomcat:9
        
        WORKDIR webapps/
        
        COPY --from=base /app/build/libs/sampleWeb-0.0.1-SNAPSHOT.war .
        
        RUN rm -rf ROOT && mv sampleWeb-0.0.1-SNAPSHOT.war ROOT.war
        ```
        
    * Here is a breakdown of what is happening in each step:
        
        1. `FROM openjdk:11 as base` - This line sets the base image for the build process to the official OpenJDK 11 image.
            
        2. `WORKDIR /app` - This line sets the working directory for the Docker container to /app.
            
        3. `COPY . .` - This line copies the current directory (where the Dockerfile is located) into the /app directory in the Docker container.
            
        4. `RUN chmod +x gradlew` - This line makes the gradlew script executable.
            
        5. `RUN ./gradlew build` - This line runs the gradlew script to build the Java application.
            
        6. `FROM tomcat:9` - This line sets the base image to the official Tomcat 9 image for the second stage of our multi stage Dockerfile
            
        7. `WORKDIR webapps/` - This line sets the working directory for the Docker container to /usr/local/tomcat/webapps, where Tomcat looks for web applications.
            
        8. `COPY --from=base /app/build/libs/sampleWeb-0.0.1-SNAPSHOT.war .` - This line copies the built Java web application from the "base" image to the current directory in the second image which is our final image.
            
        9. `RUN rm -rf ROOT && mv sampleWeb-0.0.1-SNAPSHOT.war ROOT.war` - This line removes the existing ROOT web application and renames the copied application to ROOT.war so that it becomes the default application served by Tomcat.
            
    * We can check if our docker file is working as expected or not, go to the jenkins server and navigate to the path `/var/lib/jenkins/workspace/sampleWeb_app`, here we will have all our project files from the git repo. We can create our Dockerfile here and try to build test images.
        
        ```bash
        docker build -t test-tomcat .
        
        jenkins@jenkins:~/workspace/java-gradle-app$ docker images 
        REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
        test-tomcat   latest    0589959663b8   25 minutes ago   514MB
        ```
        
    * Let's spin up a docker container from this image
        
        ```bash
        docker run -itd -p 7777:8080 test-tomcat
        
        jenkins@jenkins:~/workspace/java-gradle-app$ docker run -itd -p 7777:8080 test-tomcat
        584f79d32cfa20f8b7428cdc9d2ca80928f717e14fb93ee59ee0bd7019ce2372
        jenkins@jenkins:~/workspace/java-gradle-app$ docker ps
        CONTAINER ID   IMAGE         COMMAND             CREATED         STATUS         PORTS                                       NAMES
        584f79d32cfa   test-tomcat   "catalina.sh run"   5 seconds ago   Up 3 seconds   0.0.0.0:7777->8080/tcp, :::7777->8080/tcp   nice_mcclintock
        ```
        
    * We can access the web application by going to `jenkins_ip:7777` in our web browser.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683987992479/16b1639f-02e2-4ea3-aad1-dbc1f774b548.png align="center")
        
    * Make sure to delete the containers and images we created for our testing purpose in the previous test after confirming that our container is running as expected
        
3. **Adding Stage II : Build docker images and push to Nexus to Jenkinsfile**
    
    * We need to do four steps to build our image and push the image to the nexus repository.
        
        1. **Tag and build docker image**
            
            * We can tag our image using the command  
                `docker build -t nexus_server_ip:8083/myapp:$VERSION`
                
            * `VERSION` needs to be unique as every change to the application will trigger a new job and will create a new image so we need a variable which we can use as tag, we can use the build number of our jenkins job as a tag. The image created by the job will have the build number as tag.
                
            * For this to work, first we need to define `$VERSION` in the pipeline, it's value will be build number of the jenkins job.
                
            * Define the `VERSION` in the pipeline using environment variable which will be availabe for use during the execution of the pipeline.
                
                ```bash
                environment{
                        VERSION = "${env.BUILD_ID}"
                    }
                ```
                
        2. **Login to the Nexus repo**
            
            * This can be done using the command  
                `docker login -u admin -p $nexus_pass_var nexus_server_ip:8083`
                
            * `$nexus_pass_var` is variable through which we access the jenkins credential that we created for the storing the admin password of the nexus repository.
                
            * We will use a withCredentials block to access the credential `nexus_pass`.
                
                ```bash
                withCredentials([string(credentialsId: 'nexus_pass', variable: 'nexus_pass_var')]) {
                	sh '''
                	docker build -t nexus_server_ip:8083/myappapp:${VERSION} .
                	docker login -u admin -p $nexus_docker_repo_pass_var nexus_server_ip:8083
                ```
                
        3. **Push the docker image to the nexus repo**
            
            ```bash
            #Use this command to push the image to nexus repo
            docker push nexus_server_ip:8083/myappapp:${VERSION}
            ```
            
        4. Remove the image from the server after the image is pushed to nexus
            
            ```bash
            docker rmi nexus_server_ip:8083/myapp:${VERSION}
            ```
            
    * The final code of this stage of pipeline will look like this:  
        We've also created a new environment variable in the pipeline called `DOCKER_HOSTED_EP` which will declare the value of `nexus_machine_ip:8083` as an variable which will be available to the pipeline through all the stages.
        
        ```bash
        pipeline{
            agent any
            environment{
                VERSION = "${env.BUILD_ID}"
                DOCKER_HOSTED_EP = "13.235.91.151:8083" 
            }
            stages{
                stage("Sonar Quality Check"){
                    steps{
                        script{
                            withSonarQubeEnv(credentialsId: 'sonar-token') {
                                sh 'chmod +x gradlew'
                                sh './gradlew sonarqube --info'
                            }
                            timeout(time: 1, unit: 'MINUTES') {
                                def qg = waitForQualityGate()
                                if (qg.status != 'OK') {
                                    error "Pipeline aborted due to quality gate failure: ${qg.status}"
                                }
                            }
                        }
                    }
                }
                stage("Build docker images and push to Nexus"){
                    steps{
                        script{
                            withCredentials([string(credentialsId: 'nexus_pass', variable: 'nexus_pass_var')]) {
                                sh '''
                                docker build -t $DOCKER_HOSTED_EP/javawebapp:${VERSION} .
                                docker login -u admin -p $nexus_pass_var $DOCKER_HOSTED_EP
                                docker push $DOCKER_HOSTED_EP/javawebapp:${VERSION}
                                docker rmi $DOCKER_HOSTED_EP/javawebapp:${VERSION}
                                '''
                            }
                        }
                    }
                }
            }            
        }
        ```
        
    * Once the build is successfully completed, check the nexus repository  
        `docker-hosted` to see if the docker images were pushed to it.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683991731699/0a8e9e27-7326-47d7-8f74-e8985d4702db.png align="center")
        
    * Build was successfull, build number is **9**, let's see if we have docker images in the nexus repository with tag as **9**.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683991819474/d0cb68d9-a87d-4fdc-b116-ca46c10169ab.png align="center")
        
    * **Docker image successfully pushed to the nexus repository!!!!!**
        
    * This concludes Stage II of our Pipeline, let's move on to Stage III.
        

### STAGE III : Identify misconfigurations in Helm charts using datree.io

* In this stage we will identify the misconfigurations in our HELM charts.
    
* We have already created the required YAML files for deployments, services and helm charts. These are present in the `kubernetes/myapp` folder in the root directory of the repo.
    
* First we will need a token from our datree.io account in start analzing helm charts based on the policies/rules set in our datree.io account.
    
    1. Go to datree.io and login using your github or gmail account, after successful login, we will see the datree.io dashboard
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683992631599/16a60661-52e2-4bfa-b3ea-5e570033ea43.png align="center")
        
    2. The policies/rules against which our helm chart will be analzed can se accesed by navigating to **Policies &gt; Active Rules.**
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683993931469/b11fa144-6a3c-4ac2-b854-7f4355249c0b.png align="center")
        
    3. Next, go to **SETTINGS &gt; TOKEN MANAGEMENT,** click on `Create Token`
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683992844092/9047e966-2355-4fe7-baa3-5727c1c9a6f1.png align="center")
        
        Enter a name in the `Label` section and select `write` in the `Type`  
        section, click on Generate Token.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683992951266/02c86f37-5660-4fbe-b821-3825efe75d7b.png align="center")
        
    4. Copy the token and create a Jenkins credential datree-token of Kind Secret text, enter the token as secret, click on `Create`.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683993128323/56f89d64-9f1f-4cdf-a4e3-606f04808d49.png align="center")
        
    5. Our helm charts are in the kubernetes/myapp directory so we have to perform three steps sop that we can do static code analysis of our helm charts using datree.
        
        1. Change the working directory to `kubernetes/`
            
            ```bash
            dir('kubernetes/') {
            }
            ```
            
        2. Set datree token from the value of jenkins secret credential `datree-token`.
            
            ```bash
            dir('kubernetes/') {
                withCredentials([string(credentialsId: 'datree-token', variable: 'datree_token_var')]) {                
                    sudo helm datree config set token $datree_token_var
                }    
            }
            ```
            
        3. Run the command `helm datree test myapp/`
            
            ```bash
            dir('kubernetes/') {
                withCredentials([string(credentialsId: 'datree-token', variable: 'datree_token_var')]) {
                   sh '''
                   sudo helm datree config set token $datree_token_var
                   sudo helm datree test myapp/
                   '''
                }    
            }
            ```
            
        4. Now, the complete pipeline for this stage will look like this:
            
            ```bash
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
            
        5. Next, we wil edit the jenkins file and commit the changes to the dev branch and then start a new build.
            
            ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683995198418/1513a1b2-0f66-4ff8-b429-bdfd0ba10cee.png align="center")
            
        6. Build was successful, let's check the console output for more details.
            
            ```bash
            Pipeline] { (Identify misconfigurations in HELM charts using datree.io)
            [Pipeline] script
            [Pipeline] {
            [Pipeline] dir
            Running in /var/lib/jenkins/workspace/java-gradle-app/kubernetes
            [Pipeline] {
            [Pipeline] withCredentials
            Masking supported pattern matches of $datree_token_var
            [Pipeline] {
            [Pipeline] sh
            + sudo helm datree config set token ****
            + sudo helm datree test myapp/
            
            [K
            | Loading... WWWWWWWWWWWWWW[K[K[K[K[K[K[K[K[K[K[K[K[K[K
            [K
            / Loading... WWWWWWWWWWWWWW[K[K[K[K[K[K[K[K[K[K[K[K[K[K
            [K
            - Loading... WWWWWWWWWWWWWW[K[K[K[K[K[K[K[K[K[K[K[K[K[K
            [K
            \ Loading... WWWWWWWWWWWWWW[K[K[K[K[K[K[K[K[K[K[K[K[K[K
            [K
            | Loading... WWWWWWWWWWWWWW[K[K[K[K[K[K[K[K[K[K[K[K[K[K
            [K
            / Loading... WWWWWWWWWWWWWW[K[K[K[K[K[K[K[K[K[K[K[K[K[K
            [K
            - Loading... WWWWWWWWWWWWWW[K[K[K[K[K[K[K[K[K[K[K[K[K[K
            [K
            \ Loading... WWWWWWWWWWWWWW[K[K[K[K[K[K[K[K[K[K[K[K[K[K
            [K
            | Loading... WWWWWWWWWWWWWW[K[K[K[K[K[K[K[K[K[K[K[K[K[K
            [K
            / Loading... WWWWWWWWWWWWWW[K[K[K[K[K[K[K[K[K[K[K[K[K[K
            [K
            - Loading... WWWWWWWWWWWWWW[K[K[K[K[K[K[K[K[K[K[K[K[K[K
            [K
            \ Loading... WWWWWWWWWWWWWW[K[K[K[K[K[K[K[K[K[K[K[K[K[K
            [K
            | Loading... WWWWWWWWWWWWWW[K[K[K[K[K[K[K[K[K[K[K[K[K[K
            [K
            / Loading... WWWWWWWWWWWWWW[K[K[K[K[K[K[K[K[K[K[K[K[K[K
            [K
            - Loading... WWWWWWWWWWWWWW[K[K[K[K[K[K[K[K[K[K[K[K[K[K
            [K
            \ Loading... WWWWWWWWWWWWWW[K[K[K[K[K[K[K[K[K[K[K[K[K[K
            [K
            | Loading... WWWWWWWWWWWWWW[K[K[K[K[K[K[K[K[K[K[K[K[K[K
            [K
            / Loading... WWWWWWWWWWWWWW[K[K[K[K[K[K[K[K[K[K[K[K[K[K
            [K
            - Loading... WWWWWWWWWWWWWW[K[K[K[K[K[K[K[K[K[K[K[K[K[K
            [K
            \ Loading... WWWWWWWWWWWWWW[K[K[K[K[K[K[K[K[K[K[K[K[K[K
            [K
            | Loading... WWWWWWWWWWWWWW[K[K[K[K[K[K[K[K[K[K[K[K[K[K
            [K
            / Loading... WWWWWWWWWWWWWW[K[K[K[K[K[K[K[K[K[K[K[K[K[K
            [K
            (Summary)
            
            - Passing YAML validation: 1/1
            
            - Passing Kubernetes (1.24.0) schema validation: 1/1
            
            - Passing policy check: 1/1
            
            +-----------------------------------+-----------------------+
            | Enabled rules in policy "Starter" | 0                     |
            | Configs tested against policy     | 2                     |
            | Total rules evaluated             | 0                     |
            | [36mTotal rules skipped[0m               | [36m0[0m                     |
            | [91mTotal rules failed[0m                | [91m0[0m                     |
            | [32mTotal rules passed[0m                | [32m0[0m                     |
            | See all rules in policy           | https://app.datree.io |
            +-----------------------------------+-----------------------+
            [Pipeline] }
            [Pipeline] // withCredentials
            [Pipeline] }
            [Pipeline] // dir
            [Pipeline] }
            [Pipeline] // script
            [Pipeline] }
            [Pipeline] // stage
            [Pipeline] }
            [Pipeline] // withEnv
            [Pipeline] }
            [Pipeline] // withEnv
            [Pipeline] }
            [Pipeline] // node
            [Pipeline] End of Pipeline
            Finished: SUCCESS
            ```
            
        7. We can see that datree plugin worked as expected. This concludes Stage III.
            

### STAGE IV : Push Helm Charts to Nexus Repo

1. **Create a private repository in Nexus Repository Manager**
    
    * We will create a private repository to store our Helm charts on the Nexus Repository Manager and
        
        * Go to Nexus dashboard &gt; `Repositories` &gt; `Create Repository` Select Recipe as `helm-hosted`. Click on `Create repository`.
            
            ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683995727428/19dcd3bb-481f-4d76-bbaa-1d14a625e1e4.png align="center")
            
        * We can see our `helm-hosted` repository listed under repositories.
            
            ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683995785380/015415ab-35de-47fd-96f2-4f080019ab01.png align="center")
            
        * Next step is to create the Stage in Jenkins Pipeline for pushing the helm charts to Nexus repo, for this no other configuration is need in Jenkins host because we will use Nexus repo api to push the helm charts.
            
            ```bash
            curl -u admin:$nexus_pass_var http://nexus_machine_ip:8081/repository/helm-hosted/ --upload-file myapp-${helmchartversion}.tgz -v
            ```
            
        * We will create a new environment variable in the pipeline called `HELM_HOSTED_EP` which will replace hard coded value for `nexus_machine_ip:8081` same as when we pushed the docker images and used the environment variable `DOCKER_HOSTED_EP`.
            
        * Next, we need to package our helm according to the helm chart version, this can be done by simply running the command `helm package myapp`.
            
            ```bash
            jenkins@jenkins:~/workspace/java-gradle-app/kubernetes$ cat myapp/Chart.yaml | grep 'version:'
            version: 0.1.0
            
            jenkins@jenkins:~/workspace/java-gradle-app/kubernetes$ helm package myapp/
            
            Successfully packaged chart and saved it to: /var/lib/jenkins/workspace/java-gradle-app/kubernetes/myapp-0.1.0.tgz
            ```
            
        * Helm package command will package the helm charts accoding to the chart `version` mentioned in `myapp/Chart.yaml`.
            
        * Our final code for this stage of the pipeline will look like this
            
            ```bash
            environment{
                    VERSION = "${env.BUILD_ID}"
                    DOCKER_HOSTED_EP = "13.235.91.151:8083" 
                    HELM_HOSTED_EP = "13.235.91.151:8081"
                }
            ...
            ...
            stage("Push Helm Charts to Nexus Repo"){
               steps{
                  script{
                     dir('kubernetes/'){
                         withCredentials([string(credentialsId: 'nexus_pass', variable: 'nexus_pass_var')]) {
                               sh '''
                               helmchartversion=$(helm show chart myapp/ | grep version | awk '{print $2}')
                               helm package myapp/
                               curl -u admin:$nexus_pass_var http://$HELM_HOSTED_EP/repository/helm-hosted/ --upload-file myapp-${helmchartversion}.tgz -v
                                        '''
                                    }   
                                }
                            }
                        }
                    }
            ```
            
        * Let's run the build now.
            
            ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683997969116/a6fcf017-b27d-48d6-88a9-be8f20a18013.png align="center")
            
        * The build was successful, now let's check if there were any images pushed to the helm repository.
            
            ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683998025016/b92b4598-46b7-477e-9272-44ceb783c0ea.png align="center")
            
        * Helm Charts pushed to the helm-hosted Nexus repo with the same tag as the helm chart version.
            
        * This concludes the Stage IV of our pipeline.
            

### Stage V: Deploy application on k8s cluster

* To deploy our applicatiojn on k8s-cluster we need to perform some additional steps.
    
    1. **Configuring the Jenkins servers to access the kubernetes cluster and run administrative commands.**
        
        * This was already done by the magic of our ansible playbooks, the requirement was to install kubectl utility on the Jenkins node (implemented in `jenkins.yaml`) and copy the the kubeconfig file i.e `/etc/kubernetes/admin.conf` or `/root/.kube/config`, both of them are the same.
            
        * We've copied the `/root/.kube/config` file from the `k8s-master` node to the `$HOME` directory of the jenkins user `/var/lib/jenkins/.kube/config` on the `Jenkins` node.
            
        * These steps were performed by these plays in the `k8s_cluster_setup.yaml`.
            
            ```yaml
            #### Plays to copy kubeconfig to jenkins server for jenkins user #### Required for Stage V & VI : Deploying application on k8s cluster #####
            
            ##### PLAY 1 : COPY kubeconfig from k8s-master node to localhost #####
            -   name: Copy the kubeconfig from k8s-master 
                hosts: k8s-master
                become: true
                tasks:
                - name: Fetch the file from the k8s-master to localhost
                  run_once: yes
                  fetch:
                    src: /root/.kube/config
                    dest: buffer/
                    flat: yes
            ##### PLAY 2 : INSTALL KUBECTL, CREATE .kube direcoty in $HOME of jenkins user, copy kubeconfig from localhost buffer to Jenkins server and sets the permission to 0600 and ownership to jenkins:jenkins#####
            -   name: Jenkins User kubectl Setup
                hosts: jenkins
                become: true
                tasks:
                  - name: Install kubectl  
                    snap: 
                      name: kubectl
                      classic: true
                      state: present
                  - name: Create directory and set ownership of .kube directory for jenkins user
                    file:
                      path: /var/lib/jenkins/.kube
                      state: directory
                      owner: jenkins
                      group: jenkins
                  - name: Copy the file from localhost to jenkins
                    copy:
                      src: buffer/config
                      dest: /var/lib/jenkins/.kube/config
                      mode: "0600"
                      owner: jenkins
                      group: jenkins
            ```
            
        * We decided to add these plays to the `k8s_cluster_setup.yaml` instead of a standalone Ansible playbook because they have a dependency on the K8s cluster being set up.
            
        * If we created a standalone playbook and someone tried to run it before the cluster was provisioned, they would encounter errors. By including the plays in the `k8s_cluster_setup.yaml`, we ensure that all necessary configurations for the Jenkins node to communicate with the K8s cluster are completed after the cluster is provisioned.
            
        * We can now sit back and relax knowing that once the cluster is provisioned in the earlier stages, everything is set up correctly.
            
    2. **Adding the docker-hosted Nexus repository on both the k8s cluster nodes**
        
        1. We are using containerd (CRI) so we cannot install docker runtime and configure insecure repo for docker as it will cause many issues in our cluster.
            
        2. We will have to configure our nexus repo as an insecure repository for containerd. The below mentioned steps need to be performed on both the k8s-nodes (we are only scheduling our deployments on k8s-node1 but it's a good practice to keep the configuration consisitent so we will do it for both the nodes).
            
            1. Set the default config file for **container.d** as `/etc/containerd/config.toml`
                
                ```bash
                mkdir -p /etc/containerd
                containerd config default>/etc/containerd/config.toml
                systemctl restart containerd
                systemctl status containerd.service
                ```
                
            2. Edit the `/etc/containerd/config.toml`
                
                * Find the line `[plugins."io.containerd.grpc.v1.cri".registry.configs]`
                    
                * Add these six lines below it
                    
                    ```bash
                    [plugins."io.containerd.grpc.v1.cri".registry.configs."13.235.91.151:8083"]     
                             [plugins."io.containerd.grpc.v1.cri".registry.configs."13.235.91.151:8083".tls]
                                    ca_file = ""
                                    cert_file = ""
                                    insecure_skip_verify = true
                                    key_file = ""
                    ```
                    
                    Our docker-hosted repository is at `13.235.91.151:8083` so we are using this `IP:PORT` combination. Replace the IP address with the public ip of the Nexus Repository Manager and if you specified a different port for docker-hosted while confguring then use that.
                    
                * Find the line `[plugins."io.containerd.grpc.v1.cri".registry.mirrors]`
                    
                * Add these two lines below it
                    
                    ```bash
                    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."13.235.91.151:8083"]
                        endpoint = ["http://13.235.91.151:8083"]
                    ```
                    
                * It will look like this after all the changes.
                    
                    ```bash
                    [plugins."io.containerd.grpc.v1.cri".registry.configs]      [plugins."io.containerd.grpc.v1.cri".registry.configs."13.235.91.151:8081"]       [plugins."io.containerd.grpc.v1.cri".registry.configs."13.235.91.151:8081".tls]
                                    ca_file = ""
                                    cert_file = ""
                                    insecure_skip_verify = true
                                    key_file = ""    [plugins."io.containerd.grpc.v1.cri".registry.headers]      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
                      [plugins."io.containerd.grpc.v1.cri".registry.mirrors."13.235.91.151:8083"]
                           endpoint = ["http://13.235.91.151:8083"]
                    ```
                    
                * Restart `containerd` service and try pulling an image from the `docker-hosted` Nexus repo.
                    
                    ```bash
                    root@k8s-master:~# systemctl restart containerd.service
                    
                    root@k8s-master:~# crictl pull 13.235.91.151:8083/javawebapp:11
                    Image is up to date for sha256:603cc7300cb07bf4ec156ddfcdaa72698fbeb441bda60bbc84704505e3c1428e
                    
                    root@k8s-master:~# crictl images | grep javawebapp
                    13.235.91.151:8083/javawebapp             11                  603cc7300cb07       287MB
                    ```
                    
                * We are able to pull the container images from our private nexus repository `docker-hosted`.
                    
                * Repeat the steps on the `k8s-node1`, our application will be deployed on the worker node as by default k8s control plane node has a taint on it which forbids us to deploy any pods on it except the kube-system ones.  
                    `Taints: node-role.kubernetes.io/control-plane:NoSchedule`
                    
                * Once these steps are completed on the k8s worker node as well then we are good to go.
                    
        3. **Configure Mail Server and add post block in Jenkins Pipeline**
            
            * Goto `Manage Jankins` &gt; `Manage Plugins` &gt; `Installed` and make sure that the `Email Extension Plugin` is installed.
                
            * Goto `Manage Jankins` &gt; `Configure Systems` &gt; `Email Notification` Let's configure smtp config for our gmail account and try sending a test email.
                
                ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684005999530/890a682e-028a-47ad-9ae9-e8975367b366.png align="center")
                
            * To configure these setting we'll need to go to our gmail account settings &gt; `Manage Your Google Account`. Now we need to create an app password so that our jenkins app can send emails to our gmail account.
                
            * **Important:** To create an app password, you need 2-Step Verification on your Google Account.
                
                If you use 2-Step-Verification and get a "password incorrect" error when you sign in, you can try to use an app password.
                
                1. Go to your Google Account.
                    
                2. Select **Security**
                    
                3. Under "**Signing in to Google**," select **2-Step Verification**.
                    
                4. At the bottom of the page, select **App passwords**.
                    
                5. Enter a name that helps you remember where you’ll use the app password.
                    
                6. Select **Generate**.
                    
                7. To enter the app password, follow the instructions on your screen. The app password is the 16-character code that generates on your device.
                    
                8. Select **Done**.
                    
            * Use the created app password in the Email configuration for Jenkins (Step 2) After filling the required fields, it will look similar to this. Click on `Test Configuration`.
                
                ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684006270108/cb0a8520-0909-4b61-babf-9dec3373595b.png align="center")
                
            * Our test config is working properly, let's setup the email now.
                
                First, we have to create a credential to use the app password in the jenkins pipeline to allow access jenkins to send emails.
                
                1. Goto `Manage Jenkins` &gt; `Credentials` &gt; `select global`, select kind as `Username with password`.
                    
                    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684006423303/0805544e-ea70-43df-afd1-d22aea71b3d7.png align="center")
                    
                2. Goto `Manage Jenkins` &gt; `Configure Systems` &gt; `Extended E-mail Notification`.
                    
                    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684006597325/fbc310d1-f5f6-4db7-b258-8f8e9a51ed31.png align="center")
                    
                3. Add a post block to the jenkins pipeline after the stages block.
                    
                    ```bash
                    post {
                    	always {
                    	    mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br>Build URL: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "mandeepsingh1018@gmail.com";
                    		}
                    }
                    ```
                    
                4. Also make sure that you have filled the Email Notification section as well in addition to the Extended Email Notification, if this is missed then the build will fail.  
                    
                    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684008161716/d07f58e5-3aa6-49dc-839a-878ca8d5b163.png align="center")
                    
                5. Now build the job again, this time you'll recieve an email about the Status of the job.
                    
                    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684011233142/4f1a05bc-6a6a-4cb1-bf4c-2cae96044c47.png align="center")
                    
                    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684011250684/90810997-4385-4bc1-8eba-2e2d4997e012.png align="center")
                    
        4. **Configure Slack notifications**
            
            * Download Slack plugin
                
            * Goto `Manage Jenkins` &gt; `Manage Plugins` &gt; `Available`, search for slack and download the plugin
                
            * Login to your slack account and create a channel for jenkins notifications.
                
                ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684008339737/4a513247-a46c-4bfd-ac55-66823d2dd076.png align="center")
                
            * On next screen, select Public and then click on `Create`.  
                
                ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684008372031/c597938a-f7c1-403c-820f-30f36ba097d4.png align="center")
                
            * Goto [https://my.slack.com/services/new/jenkins-ci](https://my.slack.com/services/new/jenkins-ci)  
                
                ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684008550090/7bea0941-f210-463f-91b0-3e0f3b1a8565.png align="center")
                
            * Select the newly created slack channel for jenkins and click on `Add Jenkins CI Integration`.
                
            * Follow the on-screen instructions in step4 and get the **workspace (team subdomain)**, and **Integration Token Credential ID**
                
            * Create a secret text credential with **Integration Token Credential ID**  
                
                ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684008946212/8887e431-60a7-40a3-845c-4879e8ea087d.png align="center")
                
            * Goto `Manage Jenkins` &gt; `Configure Systems`, Search for `Slack` settings.  
                
                ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684009062229/c798a588-02ed-4241-a15f-aa0d7e869561.png align="center")
                
            * Add this to the post block in your pipeline, you can also choose to change the frequency to always, success, or failure etc, I have added it in the same always block where mail is configured.
                
                ```bash
                slackSend channel: '#jenkins-cicd', message: "Project: ${env.JOB_NAME} \n Build Number: ${env.BUILD_NUMBER} \n Status: *${currentBuild.result}* \n More info at: ${env.BUILD_URL}"
                ```
                
                ```bash
                post {
                    always {
                	  mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br>Build URL: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "mandeepsingh1018@gmail.com";
                      slackSend channel: '#jenkins-cicd', message: "Project: ${env.JOB_NAME} \n Build Number: ${env.BUILD_NUMBER} \n Status: *${currentBuild.result}* \n More info at: ${env.BUILD_URL}"
                        }
                    }  
                ```
                
            * Make sure to change the TeamDomain and channel name.
                
            * Make the changes to Jenkinsfile and push it to the dev branch.
                
            * Now, let's build the job and check if we get any slack notifications.
                
                ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684011104786/5ee9a479-8f1d-444f-9cd7-4e7b7708319d.png align="center")
                
        5. **Deploy Helm Charts on k8s cluster**
            
            1. Let's start by checking if we have any release created on our cluster.
                
                ```bash
                jenkins@jenkins:~$ helm list
                NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION
                ```
                
                We don't have any helm releases
                
            2. Let's add a new stage to our jenkins pipeline to create a release
                
                ```bash
                stage('Deploy application on k8s-cluster') {
                    steps {
                        script{
                            dir ("kubernetes/"){  
                				sh 'helm upgrade --install --set image.repository="$DOCKER_HOSTED_EP/javawebapp" --set image.tag="${VERSION}" jwa1 myapp/ ' 
                			        }   
                                }
                            }
                        }
                ```
                
                * The `steps` block contains a `script` block that changes the working directory to `kubernetes/` using the `dir` directive. This is where the Helm chart and the Kubernetes deployment configuration files are stored.
                    
                * The command is a Helm CLI command that upgrades an existing Kubernetes deployment or installs a new one using a Helm chart located in the `myapp/` directory.
                    
                * The command is broken down as follows:
                    
                    1. `helm upgrade`: This command upgrades a Helm release if it already exists, or installs a new one if it does not exist.
                        
                    2. `--install`: This flag indicates that a new release should be installed if it does not already exist.
                        
                    3. `--set`: This flag sets one or more values in the Helm chart's values file. In this case, it sets the `image.repository` and `image.tag` values to the specified values.
                        
                    4. `image.repository="13.235.91.151:8083/javawebapp"`: This sets the Docker image repository for the application to `13.235.91.151:8083/javawebapp`.
                        
                    5. `image.tag="${VERSION}"`: This sets the Docker image tag for the application to a value stored in the `${VERSION}` environment variable defined in the pipeline
                        
                    6. `jwa1`: This is the name of the Helm release.
                        
                    7. `myapp/`: This is the path to the Helm chart directory containing the `values.yaml` file and any other necessary files.
                        
            3. Create a a kubernetes secret for nexus docker-hosted repo
                
                ```bash
                kubectl create secret docker-registry registry-secret --docker-server=13.235.91.151:8083 --docker-username=admin --docker-password=msx@9797 --docker-email=not-needed@yolo.com
                ```
                
                * We need this secret to authenticate to the nexus repo hosted on our nexus server
                    
                * This is a kubectl command that creates a Kubernetes secret named `registry-secret` of type `docker-registry`. This secret is used to store credentials for authenticating with a Docker registry located at the specified IP address and port number  
                    (`13.235.91.151:8083`). The `--docker-username`, `--docker-password`, and `--docker-email` flags are used to set the corresponding authentication credentials for the Docker registry.
                    
                * When we run the commands mentioned in **step2**, the image.tag will get the value of $VERSION variable that is build number and it will also be replaced in the `values.yaml` file in helm charts directory
                    
                    ```yaml
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
                    
                * Now when the helm charts command `helm upgrade --install --set image.repository="13.235.91.151:8083/javawebapp" --set image.tag="${VERSION}" jwa myapp/` will be excuted it will replace all the values of variables, labels, annotation from the helm helper, chart.yaml and values.yaml in the deployment.yaml and services.yaml
                    
                * In deployment.yaml, in the pod template section, we have a field called `imagePullSecrets`, this field indicates that the container images for the pods are stored in a private repository.
                    
                    ```yaml
                    template:
                        metadata:
                          labels:
                            {{- include "myapp.selectorLabels" . | nindent 8 }}
                        spec:
                          imagePullSecrets:
                            - name: registry-secret
                    ```
                    
                * The secret `registry-secret` is the same secret that we created.
                    
                * Commit the Jenkinsfile changes and start a build.
                    
                    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684013083648/dbf90bf4-bd75-4fdb-9f43-5282ac5f7eb7.png align="center")
                    
                * Build was successful, let's verify the application deployment on the k8s cluster.
                    
                    ```bash
                    jenkins@jenkins:~$ kubectl get nodes
                    NAME         STATUS   ROLES           AGE   VERSION
                    k8s-master   Ready    control-plane   12h   v1.27.1
                    k8s-node1    Ready    <none>          12h   v1.27.1
                    jenkins@jenkins:~$ kubectl get deployments
                    NAME         READY   UP-TO-DATE   AVAILABLE   AGE
                    jwa1-myapp   2/2     2            2           3m59s
                    jenkins@jenkins:~$ kubectl get pods
                    NAME                          READY   STATUS    RESTARTS   AGE
                    jwa1-myapp-67cd67cbb8-n2w4x   1/1     Running   0          4m5s
                    jwa1-myapp-67cd67cbb8-s895p   1/1     Running   0          4m5s
                    jenkins@jenkins:~$ kubectl get svc
                    NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
                    jwa1-myapp   NodePort    10.100.153.6   <none>        8080:30984/TCP   4m10s
                    kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP          12h
                    jenkins@jenkins:~$ helm list
                    NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
                    jwa1	default  	1       	2023-05-13 21:24:01.463472615 +0000 UTC	deployed	myapp-0.2.0	1.16.0     
                    ```
                    
                * The deployment is up and the pods are running, let's try to access the application from the web browser using the node\_ip followed by NodePort, `35.154.247.120:30984`
                    
                    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684013445491/7358776b-a39d-426e-ad40-f450e7744a6c.png align="center")
                    
                * The java web application is up and running, we have successfully deployed our application on a k8s cluster using helm charts.
                    
        6. **Add approval request to manually approve deployments**
            
            * We can use this stage block to add a manual approval to deploy or abort the deployment.
                
                ```bash
                stage("Deployment Approval"){
                   steps{
                      script{
                           timeout(10){
                              mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> Goto : ${env.BUILD_URL} and approve/reject the deployment", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "CICD APPROVAL REQUEST: Project name -> ${env.JOB_NAME}", to: "mandeepsingh1018@gmail.com";  
                               slackSend channel: '#jenkins-cicd', message: "*CICD Approval Request* \nProject: *${env.JOB_NAME}* \n Build Number: ${env.BUILD_NUMBER} \n Status: *${currentBuild.result}* \n  Go to ${env.BUILD_URL} to approve or reject the deployment request."  
                                input(id: "DeployGate", message: "Approval required to proceed, deploy ${env.JOB_NAME}?", ok: 'Deploy')
                               }
                          }
                     }
                }
                ```
                
                ```bash
                input(id: "DeployGate", message: "Approval required to proceed, deploy ${env.JOB_NAME}?", ok: 'Deploy')
                ```
                
            * We have use an input block to register input from the approvers.
                
            * Let's push the code and build the job
                
            * We will receive email and slack notifications like this:
                
                ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684014249398/96aed27e-4580-4bc2-9ad7-84fe1f58ef59.png align="center")
                
                ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684014237991/defb988f-abac-48ba-adef-7c751bbfd154.png align="center")
                
            * We can use the build URL mentioned in the notifications to approve or reject the deployment request.
                
                ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684014404476/b2846ad1-d394-4fb0-a029-f5dc0ce2a529.png align="center")
                
            * Jenkins was waiting for us to manually Deploy or Abort the deployment request. Once we approved the request, the application was deployed to the k8s cluster.
                
                ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684014497702/f842f2b3-2297-46e1-9656-5339d1190625.png align="center")
                
            * Received slack notification for the same
                
                ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684014540408/6be70e08-4ff4-4155-9132-21ec4b10e137.png align="center")
                
            * First notification is for approving the deployment request and the second one is for the Build status update.
                
            * This concludes Stage V of our pipeline.
                

### Stage VI: Verify application deployment on k8s cluster

* ```bash
    kubectl run curl --image=curlimages/curl -i --rm --restart=Never -- curl myjavaapp-myapp:8080
    ```
    
* We can verify the application deployent using the kubectl command. We create a pod with curl image which curls the service at 8080 port and then the pod is deleted after it's work is done.
    
* If the exit code of this commad is 0 that means our application deployment was a success else it was a failure
    
* ```bash
    stage("Verify application deployment on k8s-cluster") {
        steps {
            script{
                dir ("kubernetes/"){  
    				sh 'kubectl run curl --image=curlimages/curl -i --rm --restart=Never -- curl jwa1-myapp:8080 ' 
    			        }   
                    }
                }
            } 
    ```
    
* Add this stage to the Jenkinsfile, push the code to dev branch and start the build.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684015475645/c9190bba-63e5-4642-950d-f23332e03634.png align="center")
    
* All the stages were successful. With this all the Stages of our application are now completed.
    
* One last thing is left, that is enabling pull request triggers, let's do that.
    

### Configure PR based trigger in Jenkins

* We should never make changes to the **main/master** branch of the project, we should create separate branches, develop the code and then we should create a Pull request to merge the changes of our request into the **main** branch.
    
* It is a common practice that when we raise a pull request it will automatically trigger the CICD Pipeline and then update back the status of the CI pipeline onto the pull request, because reviewers should also check who has made what changes and that the changes should not break our CICD pipeline.
    
* Let's setup PR based Triggers, install the **GitHub Pull Request Builder** plugin.
    
* Navigate to `Manage Jenkins > Configure System`, search for **GitHub Pull Request Builder** section.
    
* In this section we need to add our github creds, so go to your github account and navigate to `Settings > Developer settings > Personal access tokens`
    
    Select `Tokens (classic)`, click on `Generate new token`  
    then `Generate new token (classic)`. You will be prompted to enter your github password, enter the password and proceed.
    
* Give a name to the Token, choose expiration and then select relevant scopes.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684016853536/1e670d51-16a5-43ea-9ea6-9dfbd8138fd6.png align="center")
    
* Once done, click on **Generate Token.**
    
* Create a Jenkins credential of Kind `Username and Password` using this token as password and your github account name as username.
    
* Now, navigate to the project and go to configuration and under general, select the following things.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684017222947/7722f798-ecab-4fe9-98e5-15b31e1f07e6.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684017260806/887e16b3-b17c-4496-a050-e7db3e2ab5df.png align="center")
    
* Now go to Trigger setups.
    
* ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684017390924/609cb6ae-2bc5-4b85-bddd-10dcf1b9351e.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684017467630/0afd24e1-2750-44b3-ba8a-aa00987e790b.png align="center")
    
    Now go to the Pipeline section and provide the credentials, same one that we just created using PAT and few other settings.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684017676274/ebcf0d60-c4d0-4277-9cc1-7106f39ee286.png align="center")
    
* Next, go back to your github account, then go to the repo settings and then add a webhook `http://jenkins_public_ip:8080/ghprbhook/`
    
* Select, `Let me select individual events`, and then select issue comments, Pushes, Pull Requests.
    
* Click on Add Webhook. After some time there should be a green tick beside the webhook.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684018146559/0f7998dc-576e-4ff6-a4e4-0292260081f5.png align="center")
    
* Now let's go ahead and raise a pull request.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684018251423/5b1e0bb4-bb76-4bb8-a41e-1e3820f88bca.png align="center")
    
      
    
* Click on Create Pull Request.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684018471232/0ef674f0-21ff-4748-b43c-69f2ee5e57d7.png align="center")
    
* Next we can add reviewers, but in our case we'll be the only contributors so that's why we won't be able to add a reviewer.
    
* Next click on **Create Pull Request**. As soon as we create a pull request a trigger has happened.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684018589329/525bc86c-67f9-4c8b-add8-925180f3f422.png align="center")
    
* A jenkins build was triggered.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684018835416/e30f590b-6beb-4cb1-b764-9bfd0b7c321b.png align="center")
    
* Our build was successful so now we can go ahead and merge the pull request.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684019084674/44d0dcb6-71f1-427b-bb44-7b668cc61fc9.png align="center")
    
* Now our dev branch was merged into Main branch, so we don't need the dev branch, we can safely delete it.
    
* Now we can see that all the updated config files from the dev branch are available in the Main branch.
    
* ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684019162984/247c866b-f55e-4ca9-920c-0f9a30b6f48a.png align="center")
    
* We have successly replicated an advanced end-to-end DevOps pipeline.
    

### THE END.