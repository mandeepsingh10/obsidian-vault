
1. Download Slack plugin 
  - Goto `Manage Jenkins` > `Manage Plugins` > `Available`, search for slack and download the plugin

2. Login to your slack account and create a channel for jenkins notifiactions

3. Goto [https://my.slack.com/services/new/jenkins-ci](https://my.slack.com/services/new/jenkins-ci)
4. Follow the on-screen instructions in step4 and get the **workspace (team subdomain)**, and **Integration Token Credential ID**
5. Create a secret text credential with **Integration Token Credential ID**
6. Goto `Manage Jenkins` > `Configure Systems`, Search for `Slack` settings.
	 ![[Screenshot from 2023-05-11 13-21-25.png]]

7. Add a post block in your pipeline, example
```Shell
post {
	always {
		slackSend channel: '#jenkins-bot', message: "Project: ${env.JOB_NAME} \n Build Number: ${env.BUILD_NUMBER} \n Status: *${currentBuild.result}* \n More info at: ${env.BUILD_URL}"
		}
}
```

8. This post block will send a notification like this on your slack channel.
	 ![[Pasted image 20230511142016.png]]