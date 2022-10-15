pipeline {
    agent {
        label 'linux2'
    } 
    environment {
        DOCKER_USERNAME     = credentials('jenkins-docker-username')
        DOCKER_PASSWORD     = credentials('jenkins-docker-password')
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
    }
    stages {
        stage('Testing Image') { 
            steps {
                sh 'echo "Testing Image by building container"'
                sh '''
                    docker run -d --rm -it -p 80$BUILD_NUMBER:80 --name webserver-$BUILD_NUMBER jhadeepak/jenkins:$BUILD_NUMBER
                    URL="http://localhost:80$BUILD_NUMBER"
                    STATUS=`curl $URL -k -s -f -o /dev/null && echo "SUCCESS" || echo "ERROR"`
                    if [[ $STATUS != SUCCESS ]]; then
                        exit 1
                    fi
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
        stage('Cleanup') { 
            steps {
                sh 'echo "Stoping Container and removing image"'
                sh '''
                    docker stop webserver-$BUILD_NUMBER
                    docker rmi jhadeepak/jenkins:$BUILD_NUMBER --force
                    
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