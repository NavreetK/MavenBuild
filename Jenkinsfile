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
        TOMCAT_CREDENTIALS = credentials('tomcat')
        WAR_FILE = 'target/1.0-SNAPSHOT/*.war'
       // CONTEXT_PATH = 'your-app-context-path' 
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
        stage('Download WAR File') {
            steps {
                script {
                    def nexusUrl = "http://15.222.102.23:8081/repository/maven-repo/com/nexus/MavenBuild/1.0-SNAPSHOT/maven-metadata.xml"
                    def response = sh(script: "curl -s ${nexusUrl}", returnStdout: true).trim()
                    def warFileName = sh(script: "echo ${response} | grep -oPm1 '(?<=<latest>)[^<]+'", returnStdout: true).trim()
                    def warUrl = "http://15.222.102.23:8081/repository/maven-repo/com/nexus/MavenBuild/1.0-SNAPSHOT/${warFileName}"
                    
                    sh(script: "curl -o Maven-Nexus-Build.war ${warUrl}")
                }
            }
        }
        
        stage('Deploy to Tomcat') {
    steps {
        script {
            
           def tomcatHome = "/opt/tomcat/"
                    def warFilePath = "Maven-Nexus-Build.war"  // Adjust the path if needed

                    sh(script: "${tomcatHome}/bin/shutdown.sh")
                    sh(script: "rm -rf ${tomcatHome}/webapps/Maven-Nexus-Build*")
                    sh(script: "cp ${warFilePath} ${tomcatHome}/webapps/")
                    sh(script: "${tomcatHome}/bin/startup.sh")
        }
    }
}
    
    }
}
