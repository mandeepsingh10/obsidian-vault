
# AWS Project
Migrated a monolithic knowledge base application to AWS cloud across three availability zones by migrating the database to an RDS instance, local storage to an EFS volume. Implemented autoscaling to handle peak traffic and deployed the application across three availability zones for high availability.


# CICD Project 

Developed and executed an end-to-end CICD pipeline utilizing Jenkins to deploy a Java web application. Employed a range of tools at each stage of the pipeline, including SonarQube for code analysis, Docker for containerizing the application, Sonatype Nexus to store Docker images in a private repository, and Helm charts and datree.io for identifying misconfigurations in the charts before deploying the application to the Kubernetes cluster. Utilized Terraform and Ansible to provision and configure AWS resources required for establishing the infrastructure necessary for the CICD pipeline, such as Jenkins server, SonarQube server, Sonatype Nexus server for hosting private Docker and Helm repositories, and a Kubernetes cluster for deploying the application.