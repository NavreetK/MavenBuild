pipeline {
    agent { label 'jenkins-node'}
    tools {
        maven "Maven"
        jdk "Java9"
    }

    environment {
        // This can be nexus3 or nexus2
        NEXUS_VERSION = "nexus3"
        // This can be http or https
        NEXUS_PROTOCOL = "http"
        // Where your Nexus is running
        NEXUS_URL = "15.222.102.23:8081"
        // Repository where we will upload the artifact
        NEXUS_REPOSITORY = "maven-repo"
        // Jenkins credential id to authenticate to Nexus OSS
        NEXUS_CREDENTIAL_ID = "nexus_ID"

        TOMCAT_URL = 'http://35.182.165.190:8080'
        TOMCAT_CREDENTIALS = "tomcat"
        WAR_FILE = 'http://15.222.102.23:8081/repository/maven-repo/com/nexus/MavenBuild/1.0-SNAPSHOT/MavenBuild-1.0-20230923.175609-1.war'
       CONTEXT_PATH = 'maven-repo' 
        // Optional, defaults to the name of the war file
        
    }

    stages {
        stage("Check out") {
            steps {
                script {
                     git credentialsId: 'github', branch: 'nexus', url: 'https://github.com/NavreetK/MavenBuild'
                }
            }
        }

        stage("mvn build") {
            steps {
                script {
                    sh "mvn clean package"
                }
            }
        }

        stage("publish to nexus") {
            steps {
                script {
                    // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
                    pom = readMavenPom file: "pom.xml";
                    // Find built artifact under target folder
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    // Print some info from the artifact found
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    // Extract the path from the File found
                    artifactPath = filesByGlob[0].path;
                    // Assign to a boolean response verifying If the artifact name exists
                    artifactExists = fileExists artifactPath;

                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";

                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                // Artifact generated such as .jar, .ear and .war files.
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging]
                            ]
                        );

                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }
        stage('Deploy to Tomcat') {
    steps {
        script {
            def tomcatHome = "/opt/tomcat/"
            
            def username = TOMCAT_CREDENTIALS.username
            def password = TOMCAT_CREDENTIALS.password
            
            sh(script: "${tomcatHome}/bin/shutdown.sh")
            sh(script: "rm -rf ${tomcatHome}/webapps/${CONTEXT_PATH}*")
            sh(script: "cp ${WAR_FILE} ${tomcatHome}/webapps/${CONTEXT_PATH}.war")
            sh(script: "${tomcatHome}/bin/startup.sh")
            
            // Deploying using Tomcat Manager API (assuming manager-script role is configured in Tomcat)
            sh(script: "${tomcatHome}/bin/curl -u ${username}:${password} http://http://15.222.102.23:8080/manager/text/undeploy?path=/${CONTEXT_PATH}")
            sh(script: "${tomcatHome}/bin/curl -u ${username}:${password} http://http://15.222.102.23:8080/manager/text/deploy?path=/${CONTEXT_PATH}&war=file:${tomcatHome}/webapps/${CONTEXT_PATH}.war")
        }
    }
}

    
    
    }
}
