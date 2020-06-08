pipeline {
    agent any
	   tools {
        // Note: This should match with the tool name configured in your jenkins instance (JENKINS_URL/configureTools/)
        maven "apache-maven-3.6.3" 
    }
    environment{
        DOCKER_TAG=getDockerTag()
       
    }
    stages{
	    stage("mvn build") {
            steps {
                script {
                    // If you are using Windows then you should use "bat" step
                    // Since unit testing is out of the scope we skip them
                    sh "mvn package -DskipTests=true"
                }
            }
        }
        stage('Build Docker Image'){
            steps{
                sh "docker build . -t raouagara/spring-boot-mongo-docker:${DOCKER_TAG}"
            }
        }
	     stage('Docker push image'){
            steps{
		    withCredentials([string(credentialsId: 'DOCKER_HUB_CREDENTIALS', variable: 'DOCKER_HUB_CREDENTIALS')]) {
			    sh "docker login -u raouagara -p ${DOCKER_HUB_CREDENTIALS}" 
			    sh "docker push raouagara/spring-boot-mongo-docker:${DOCKER_TAG}"
    }
	    }
	     }
   

	    stage('Deploy app') {
     steps {
    
               kubernetesDeploy(
		       configs: 'springBootMongo.yml',
		       kubeconfigId: 'mykubeconf',
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
