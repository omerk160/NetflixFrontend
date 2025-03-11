pipeline {
    agent {
        label 'general'
    }

    triggers {
        githubPush()
    }

    options {
        timeout(time: 10, unit: 'MINUTES')
        timestamps()
    }

    environment {
        GIT_COMMIT = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        TIMESTAMP = new Date().format("yyyyMMdd-HHmmss")

        IMAGE_TAG = "v1.0.$BUILD_NUMBER"
        IMAGE_BASE_NAME = "netflix-frontend"

        DOCKER_CREDS = credentials('dockerhub')  // Correct credential ID
        DOCKER_USERNAME = "${DOCKER_CREDS_USR}"  // Use the credentials values correctly
        DOCKER_PASS = "${DOCKER_CREDS_PSW}"
    }

    stages {
        stage('Docker setup') {
            steps {
                sh '''
                    docker login -u $DOCKER_USERNAME -p $DOCKER_PASS
                '''
            }
        }

        stage('Build & Push') {
            steps {
                sh '''
                    IMAGE_FULL_NAME=$DOCKER_USERNAME/$IMAGE_BASE_NAME:$IMAGE_TAG
                    docker build --push -t $IMAGE_FULL_NAME .
                '''
            }
        }

        stage('Trigger Deploy') {
            steps {
                build job: 'NetflixFrontendDeploy', wait: false, parameters: [
                    string(name: 'SERVICE_NAME', value: 'NetflixFrontend'),
                    string(name: 'IMAGE_FULL_NAME_PARAM', value: "$DOCKER_USERNAME/$IMAGE_BASE_NAME:$IMAGE_TAG")
                ]
            }
        }
    }
}
