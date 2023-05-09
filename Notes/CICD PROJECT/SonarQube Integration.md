
####
1. Go to your Jenkins Instance and create a pipeline job with SCM as git and add the project repository, create a new branch `devops` for the project and specify this new branch in branch specifier.
2. Clone the project repo locally and add a Jenkins file to it.
3. Install sonarqube jenkins plugins and also docker build plugins which we'll also require.
	   - SonarQube Scanner for Jenkins
	   - Sonar Gerrit Plugin
	   - Sonar Quality Gates Plugin
	   - SonarQube Generic Coverage Plugin
	   - Quality Gates Plugin
	   - Docker
	   - Docker Pipeline
	   - docker-build-step
1. Goto `Manage Jenkins` > `Configure System` . There will be a section for SonarQube and we need to update the details there.
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
	![[Pasted image 20230508232732.png]]
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
 