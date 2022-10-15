pipeline {
    agent any 
    environment {
        DOCKER_USERNAME     = credentials('jenkins-docker-username')
        DOCKER_PASSWORD     = credentials('jenkins-docker-username')
	}
    stages {
        stage('Building Image') { 
            steps {
                sh 'echo "Building docker image"'
                sh '''
                    docker build -t jhadeepak/jenkins:$BUILD_NUMBER .
                '''
            }
        }
        stage('Pushing Image') { 
            steps {
                sh 'echo "Pusing Image to registry"'
                sh '''
                    docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
                    docker push jhadeepak/jenkins:$BUILD_NUMBER
                '''
            }
        }
        
    }
	post {
        always {
                   
            emailext body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}",
            recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']],
            subject: "Jenkins Build ${currentBuild.currentResult}: Job ${env.JOB_NAME}"
        }
    }
}