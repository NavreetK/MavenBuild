pipeline{
    agent any
    tools{
        maven 'Maven'
	    jdk "Java9"
    }
    stages{
        stage('Git Checkout'){
            steps{
                git credentialsId: 'github', branch: 'nexus', url: 'https://github.com/NavreetK/MavenBuild'
            }
        }
        stage('Maven build'){
            steps{
                sh 'mvn clean package'
            }
        }
        stage('Publish to Nexus'){
            steps{
             nexusArtifactUploader artifacts: [
                 [
                     artifactId: 'MavenBuild', 
                     classifier: '', 
                     file: 'target/MavenBuild-0.1.war', 
                     type: 'war'
                ]
            ], 
            credentialsId: 'nexus_ID', 
            groupId: 'in.javahome', 
            nexusUrl: '15.222.102.23:8081', 
            nexusVersion: 'nexus3', 
            protocol: 'http', 
            repository: 'MavenBuild', 
            version: '1.0'
            }
        }
        stage('Deploy Tomcat'){
            steps {
                sshagent(['tomcat_PEM']){
                    sh """
                        scp -o StrictHostKeyChecking=no target/*.war ubuntu@3.99.128.51:/opt/tomcat/webapps
                        ssh -o StrictHostKeyChecking=no ubuntu@3.99.128.51 /opt/tomcat/bin/startup.sh
			    		
		             """   
                }
            }
        }
    }
}
