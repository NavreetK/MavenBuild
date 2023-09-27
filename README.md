# MavenBuild
Description: This project is for Jenkins integration with Maven, Nexus, and Tomcat.
Creating a Maven project successfully, publishing it to Nexus and then deploy that project to Tomcat.

See screenshots below for reference:

1. AWS Instances for Jenkins, slave, Nexus and Tomcat
Using t2.micro Instance is not advised for this project- to avoid project crashing
t2.medium or higher is recommended
![Instances](https://github.com/NavreetK/MavenBuild/blob/nexus/Photos/awsInstances.PNG)

2. Nexus repository used to store the Maven project
![Nexus](https://github.com/NavreetK/MavenBuild/blob/nexus/Photos/nexus.PNG)

3. Successful Jenkins build - Maven projected created and published to Nexus
![Jenkins Build](https://github.com/NavreetK/MavenBuild/blob/nexus/Photos/jenkinsBuild.PNG)

4. Tomcat Manager 
![Tomcat](https://github.com/NavreetK/MavenBuild/blob/nexus/Photos/tomcatManager.PNG)

5. Jenkins Final Build - The Maven project successfully deployed to Tomcat
![Jenkins Final Build](https://github.com/NavreetK/MavenBuild/blob/nexus/Photos/jenkinsfinalbuild.PNG)

6. Maven Project Deployed to Tomcat Successfully
![Project Deployed](https://github.com/NavreetK/MavenBuild/blob/nexus/Photos/tomcatdeployed.PNG)


NOTES: It is advised to use same .pem key for Jenkins and Tomcat Instances to avoid the Public Key Permission errors.





