
####
1. Clone the project repo and create a new branch called devops in the project 
2. Go to your Jenkins Instance and create a pipeline job with SCM as git and add the project repository, create a new branch `devops` for the project and specify this new branch in branch specifier.
     ![[Pasted image 20230511091609.png]]
3. 
4. Install sonarqube jenkins plugins and also docker build plugins which we'll also require.
	   - SonarQube Scanner for Jenkins
	   - Sonar Gerrit Plugin
	   - Sonar Quality Gates Plugin
	   - SonarQube Generic Coverage Plugin
	   - Quality Gates Plugin
	   - Docker
	   - Docker Pipeline
	   - docker-build-step
5. Goto `Manage Jenkins` > `Configure System` . There will be a section for SonarQube and we need to update the details there.
	![[Pasted image 20230508225116.png]]
We need to create a token so that Jenkins can authenticate sonarqube.

5. Goto SonarQube app,  goto `Administration` > `Security`. Now generate a token for administrator user.
	![[Pasted image 20230508225442.png]]

6. Now copy this token and go to Jenkins application. 
    Goto `Manage Jenkins` > `Security` > `Credentials`. 
    Click on `global` and then `add credentials`
    ![[Pasted image 20230508230050.png]]

7. Now go back to step4 and configure the sonarqube details.
	![[Pasted image 20230508230315.png]]

8. Goto the pipeline job and click on `pipeline syntax` and generate sonarqube pipeline script from the snippet generator
	![[Pasted image 20230508230542.png]]

9. Now add this pipeline script to the Jenkins file under Stage `Sonar Quality Check`
	![[Pasted image 20230511095042.png]]
	 We have give execute permission to gradlew and then we have executed 
	 `./gradlew sonarqube` which helps us in pushing the code to sonarQube, where we will validate our check against the sonar rules.

10. Now add the JenkinsFile to the git repo and push the changes.
	```Shell
CICD_Java_gradle_application on  devops  🛤️ ×1via 🅶 v7.1.1 via ☕ 
➜ git add .
CICD_Java_gradle_application on  devops via 🅶 v7.1.1 via ☕ 
➜ git push --set-upstream origin devops
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 8 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 466 bytes | 466.00 KiB/s, done.
Total 3 (delta 1), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
remote: 
remote: Create a pull request for 'devops' on GitHub by visiting:
remote:      https://github.com/mandeepsingh10/CICD_Java_gradle_application/pull/new/devops
remote: 
To github.com:mandeepsingh10/CICD_Java_gradle_application.git
 * [new branch]      devops -> devops
Branch 'devops' set up to track remote branch 'devops' from 'origin'.	
```

	We can see the same on our github repo page.
	![[Pasted image 20230508231818.png]]

11. Next, Let's go to the Jenkins app and build the job.
12. Now we also need to create a webhook to have connection between jenkins and sonarqube.
	  - To create a webhook navigate to sonarqube portal then `administration --> configuration --> webhooks `
	  - Enter the URL of your jenkins host with port suffixed with ``/sonarqube-webhook/
	  - The URL will be `http://jenkins_ip:8080/sonarqube-webhook/`
	   ![[Pasted image 20230511095912.png]]

	- The data that will be sent to jenkins via the sonarqube webhook can be seen by clicking the small icon beside the time in the `Last delivery` column, see below screenshot.
		 ![[Pasted image 20230511100445.png]]
	- We are interested in the qualityGate section of the `Last Delivery` data.
		![[Pasted image 20230511100546.png]]
		```Shell
		"qualityGate": {
		    "name": "Sonar way",
		    "status": "OK",
		```
	-  If the status is `ok` then only our pipeline will move on to the next stage otherwise the build will fail.

13. Now we will add a block to our `Sonar Quality Check` stage which will wait for 15 minutes for the Quality Gate status to chaneg to `ok` state.
```Shell
timeout(time: 15, unit: 'MINUTES') {
    def qg = waitForQualityGate()
    if (qg.status != 'OK') {
        error "Pipeline aborted due to quality gate failure: ${qg.status}"
        }
    }		
```
14. Our Final Jenkinsfile will look like this
		![[Pasted image 20230511101905.png]]
15.  Now let's run the build.
	   ![[Pasted image 20230511102033.png]]