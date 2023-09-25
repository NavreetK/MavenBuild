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

       	    TOMCAT_URL = 'http://3.98.131.194:8080/manager'
        TOMCAT_USERNAME = 'admin'
        TOMCAT_PASSWORD = 'admin123'
        WAR_FILE_PATH = 'target/*.war'
        CONTEXT_PATH = '/myapp'
        
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

        stage('Deploy to Tomcat'){
            steps {
                script {
                    def response = sh(script: "curl -u ${TOMCAT_USERNAME}:${TOMCAT_PASSWORD} ${TOMCAT_URL}/list", returnStdout: true).trim()
                    if (response.contains(CONTEXT_PATH)) {
                        echo "Undeploying existing application..."
                        sh "curl -u ${TOMCAT_USERNAME}:${TOMCAT_PASSWORD} ${TOMCAT_URL}/undeploy?path=${CONTEXT_PATH}"
                    }
                    
                    echo "Deploying application..."
                    sh "curl -u ${TOMCAT_USERNAME}:${TOMCAT_PASSWORD} ${TOMCAT_URL}/deploy?path=${CONTEXT_PATH}&war=file:${WAR_FILE_PATH}"
                }
            }
        }
       
    }
}
