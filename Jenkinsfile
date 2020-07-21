pipeline {

    agent {
        label "master"
    }

    tools {
        // Note: This should match with the tool name configured in your jenkins instance (JENKINS_URL/configureTools/)
        maven "apache-maven-3.6.3" 
    }

    environment {
        // This can be nexus3 or nexus2
        NEXUS_VERSION = "nexus3"
        // This can be http or https
        NEXUS_PROTOCOL = "http"
        // Where your Nexus is running. 'nexus-3' is defined in the docker-compose file
        NEXUS_URL = "isetch:8081"
        // Repository where we will upload the artifact
        NEXUS_REPOSITORY = "CICDtest"
        // Jenkins credential id to authenticate to Nexus OSS
        NEXUS_CREDENTIAL_ID = "nexus-credentials"
        
        DOCKER_TAG=getDockerTag()
    }

    stages {
        stage("clone code") {
            steps {
                script {
                    // Let's clone the source
                    git 'https://github.com/Raouagarati101/spring-boot-mongo-docker.git';
                }
            }
        }
        stage("mvn build") {
            steps {
                script {
                    // If you are using Windows then you should use "bat" step
                    // Since unit testing is out of the scope we skip them
                    sh "mvn package -DskipTests=true"
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
                          //  Artifact generated such as .jar, .ear and .war files.
                            [artifactId: pom.artifactId,
                              classifier: '',
                                file: artifactPath,
                                type: pom.packaging],

                                // Lets upload the pom.xml file for additional information for Transitive dependencies
                             [artifactId: pom.artifactId,
                              classifier: '',
                              file: "pom.xml",
                             type: "pom"]
                            ]
                        );

                  } else {
                       error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
      }

stage("Building SONAR") {
    steps {
        script {
           def sonarUrl = 'sonar.host.url=http://isetch:9000'
         def mvnHome =  tool name: 'apache-maven-3.6.3', type: 'maven'
        withSonarQubeEnv('sonarqube') { 
          sh "mvn sonar:sonar"
  
        }
          }
    }
}
        
    stage('Build Docker Image'){
            steps{
                sh "docker build . -t raouagara/spring-boot-mongo:${DOCKER_TAG}"
            }
        }
	     stage('Docker push image'){
            steps{
		    withCredentials([string(credentialsId: 'DOCKER_HUB_CREDENTIALS', variable: 'DOCKER_HUB_CREDENTIALS')]) {
			    sh "docker login -u raouagara -p ${DOCKER_HUB_CREDENTIALS}" 
			    sh "docker push raouagara/spring-boot-mongo:${DOCKER_TAG}"
    }
    }
    }
	 stage('Deploy app on k8S'){
     steps{
               kubernetesDeploy(
		       configs: 'springBootMongo.yml',
		       kubeconfigId: 'mykubeconfig',
		       enableConfigSubstitution: true
		       )
	    }    
	}
	     }
  }
def getDockerTag() {
		def tag = sh script: 'git rev-parse HEAD', returnStdout: true
		return tag 
	} 
