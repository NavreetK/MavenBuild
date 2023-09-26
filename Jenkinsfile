pipeline {
    agent any
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
            steps{
             nexusArtifactUploader artifacts: [
                 [
                     artifactId: 'MavenBuild', 
                     classifier: '', 
                     file: 'target/MavenBuild-1.0-SNAPSHOT.war', 
                     type: 'war'
                ]
            ], 
            credentialsId: 'nexus_ID', 
            groupId: 'com.nexus', 
            nexusUrl: '15.222.102.23:8081', 
            nexusVersion: 'nexus3', 
            protocol: 'http', 
            repository: 'MavenBuild', 
            version: '1.0'
            }
        }

        stage('Deploy to Tomcat'){
            steps {

                sshagent(['tomcat_PEM']){
                    sh """
                        scp -o StrictHostKeyChecking=no target/*.war ubuntu@3.99.128.51:/opt/tomcat/webapps
                        ssh -o StrictHostKeyChecking=no ubuntu@3.99.128.51 /opt/tomcat/bin/startup.sh
			    		
		             """   
                }
                /*
                script {
                    def response = sh(script: "curl -u ${TOMCAT_USERNAME}:${TOMCAT_PASSWORD} ${TOMCAT_URL}/list", returnStdout: true).trim()
                    if (response.contains(CONTEXT_PATH)) {
                        echo "Undeploying existing application..."
                        sh "curl -u ${TOMCAT_USERNAME}:${TOMCAT_PASSWORD} ${TOMCAT_URL}/undeploy?path=${CONTEXT_PATH}"
                    }
                    
                    echo "Deploying application..."
                    sh "curl -u ${TOMCAT_USERNAME}:${TOMCAT_PASSWORD} ${TOMCAT_URL}/deploy?path=${CONTEXT_PATH}&war=file:${WAR_FILE_PATH}"
                }
                */
            }
        }
       
    }
}
