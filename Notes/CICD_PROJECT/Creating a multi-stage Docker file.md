
```Dockerfile
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


This Dockerfile is creating a Docker image that deploys a Java web application to Tomcat using Docker multi-stage build. Here is a breakdown of what is happening in each step:

1.  `FROM openjdk:11 as base` - This line sets the base image for the build process to the official OpenJDK 11 image.
    
2.  `WORKDIR /app` - This line sets the working directory for the Docker container to /app.
    
3.  `COPY . .` - This line copies the current directory (where the Dockerfile is located) into the /app directory in the Docker container.
    
4.  `RUN chmod +x gradlew` - This line makes the gradlew script executable.
    
5.  `RUN ./gradlew build` - This line runs the gradlew script to build the Java application.
    
6.  `FROM tomcat:9` - This line sets the base image to the official Tomcat 9 image for the second stage of our multi stage Dockerfile 
    
7.  `WORKDIR webapps/` - This line sets the working directory for the Docker container to /usr/local/tomcat/webapps, where Tomcat looks for web applications.
    
8.  `COPY --from=base /app/build/libs/sampleWeb-0.0.1-SNAPSHOT.war .` - This line copies the built Java web application from the "base" image to the current directory in the second image which is our final image.
    
9.  `RUN rm -rf ROOT && mv sampleWeb-0.0.1-SNAPSHOT.war ROOT.war` - This line removes the existing ROOT web application and renames the copied application to ROOT.war so that it becomes the default application served by Tomcat.


- We can check if our docker file is working as expected or not, go to the the jenkins server and go to the path `/var/lib/jenkins/workspace/Java_gradle_application`, here we will have all our project files from the git repo. We can create our Dockerfile here and test.
  
```Shell
# cd to /var/lib/jenkins/workspace/Java_gradle_application
#Create the Dockerfile as given above and run the follwing command
docker build -t tomcat-test .
```

![[Pasted image 20230511110140.png]]

- Check the created images on the jenkins server
```Shell
jenkins@jenkins:~/workspace/Java_gradle_application$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED              SIZE
tomcat-test   latest    bd968d68fe63   About a minute ago   514MB
openjdk       11        47a932d998b7   9 months ago         654MB
```

- Now let's spin up a docker container from this image
```Shell
## Running a docker container which redirects any traffic on the host 7777 port to 8080 port on the container, we did this because tomcat's default port is 8080 but we already have our jenkins server running on that port.
##
docker run -itd -p 7777:8080 tomcat-test
```

![[Pasted image 20230511110726.png]]

- We can see the webpage by going to `jenkins_ip:7777` in our browser.
   ![[Pasted image 20230511115543.png]]
- Remember to delete the container and images we created for our testing purpose in the previous test after confirming that our container is running as expected.
   