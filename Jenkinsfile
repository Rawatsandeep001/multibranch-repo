
pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

    environment {
        IMAGE_NAME = 'sandeeprawat001/multibrach-flask-app'
        GIT_USER = 'sandeeprawat001'
        GIT_EMAIL = 'sandeeprawat25691@gmail.com'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Push Image') {
            when {
                branch 'main'
            }
            steps {
                script {
                    env.IMAGE_TAG = "build-${env.BUILD_NUMBER}"

                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh '''
                            set -e
                            docker build -t "${IMAGE_NAME}:${IMAGE_TAG}" .
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            docker push "${IMAGE_NAME}:${IMAGE_TAG}"
                            docker logout
                        '''
                    }
                }
            }
        }

        stage('Update K8s Manifest') {
            when {
                branch 'main'
            }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github-creds',
                    usernameVariable: 'GIT_USERNAME',
                    passwordVariable: 'GIT_TOKEN'
                )]) {
                    sh '''
                        set -e

                        git config user.name "$GIT_USER"
                        git config user.email "$GIT_EMAIL"

                        git fetch origin
                        git checkout main
                        git reset --hard origin/main

                        sed -i "s|^[[:space:]]*image:.*|          image: ${IMAGE_NAME}:${IMAGE_TAG}|" k8s/deployment.yml

                        git add k8s/deployment.yml

                        git diff --cached --quiet || git commit -m "Updated Image to ${IMAGE_TAG}"

                        git -c credential.helper= \
                            -c core.askPass=/bin/false \
                            push "https://${GIT_USERNAME}:${GIT_TOKEN}@github.com/rawatsandeep001/multibranch-repo.git" main
                    '''
                }
            }
        }
    }
}
