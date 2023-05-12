
- We can use this stage block to add a manual approval to deploy or abort the deployment.

```Shell
stage(Deployment Approval){
            steps{
                scripts{
                    timeout(10){
                          mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> Goto : ${env.BUILD_URL} and approve/reject the deployment", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "CICD APPROVAL REQUEST: Project name -> ${env.JOB_NAME}", to: "mandeepsingh1018@gmail.com";  
                          slackSend channel: '#jenkins-bot', message: "*CICD Approval Request* \nProject: *${env.JOB_NAME}* \n Build Number: ${env.BUILD_NUMBER} \n Status: *${currentBuild.result}* \n  Go to ${env.BUILD_URL} to approve or reject the deployment request."  
                          input(id: "DeployGate", message: "Approval required to proceed, deploy ${env.JOB_NAME}?", ok: 'Deploy')
                    }
                }
            }
        }
```

```Shell
input(id: "DeployGate", message: "Approval required to proceed, deploy ${env.JOB_NAME}?", ok: 'Deploy')
```

- We have use an input block to register input from the approvers.
  
- We will receive email and slack notifications like this
	  ![[Pasted image 20230511225848.png]]

	 ![[Pasted image 20230511225946.png]]

 - We can use the build URL mentioned in the notifications to approve or reject the deployment request.
	   ![[Pasted image 20230511230108.png]]
 - The approval request has timer of 10 minutes after that it will be automatically rejected.
 