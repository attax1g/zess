pipeline {

    agent {
        label "master"
    }

    tools {
        // Note: This should match with the tool name configured in your jenkins instance (JENKINS_URL/configureTools/)
        maven "maven_3_6_3" 
    }

    stages {
        stage("clone code") {
            steps {
                script {
                    // Let's clone the source
                    git 'https://github.com/attax1g/zess.git';
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
        
    }
}
