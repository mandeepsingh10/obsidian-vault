
1. Goto `Manage Jankins` > `Manage Plugins` > `Installed` and make sure that the 
   `Email Extension Plugin` is installed.
	![[Pasted image 20230509170304.png]]

2. Goto `Manage Jankins` > `Configure Systems` > `Email Notification`
	 Let's try sending a test email and configure smtp config for our gmail account
	 ![[Pasted image 20230509170643.png]]

	To configure these setting we'll need to go to our gmail account settings > `Manage Your Google Account`. Now we need to create an app password so that our jenkins app can send emails to our gmail account.


3. **Important:** To create an app password, you need 2-Step Verification on your Google Account.
	
	If you use 2-Step-Verification and get a "password incorrect" error when you sign in, you can try to use an app password.
	
	1.  Go to your Google Account.
	2.  Select **Security**
	3.  Under "Signing in to Google," select **2-Step Verification**.
	4.  At the bottom of the page, select **App passwords**.
	5.  Enter a name that helps you remember where you’ll use the app password.
	6.  Select **Generate**.
	7.  To enter the app password, follow the instructions on your screen. The app password is the 16-character code that generates on your device.
	8.  Select **Done**.

4. Now use the created app password in the Email configuration for Jenkins (Step 2)
    After filling the required fields, it will look similar to this.
    ![[Pasted image 20230509172004.png]]

    Now our test config is working properly, let's setup the email now.

5. Goto `Manage Jankins` > `Configure Systems` > `Extended E-mail Notification`

	![[Pasted image 20230509172913.png]]

6. Add a post block to the jenkins pipeline after the stages block.
```
post {
	always {
	mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "mandeepsingh1018@gmail.com";

		}

}	
```

7. Add the file to git repo, commit and push the changes.
```
CICD_Java_gradle_application on  devops via 🅶 v7.1.1 via ☕ 
➜ git add .
CICD_Java_gradle_application on  devops  🗃️×1via 🅶 v7.1.1 via ☕ 
➜ git commit -m "Added email configuration"
[devops 574ee89] Added email configuration
 1 file changed, 6 insertions(+)
CICD_Java_gradle_application on  devops  🏎️  💨 ×1via 🅶 v7.1.1 via ☕ 
➜ git push
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 8 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 553 bytes | 553.00 KiB/s, done.
Total 3 (delta 2), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (2/2), completed with 2 local objects.
```

8. Now build the job again, this time you'll recieve an email about the Status of the job.
	![[Pasted image 20230509175225.png]]