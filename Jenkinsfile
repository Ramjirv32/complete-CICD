pipeline {
    agent any

    environment {
        SONAR_HOST_URL  = 'https://lynx-fun-normally.ngrok-free.app'
        SONAR_TOKEN     = 'sqp_15da2dada419712d578bc42619572ae7f5168f03'
        DOCKERHUB_USER  = 'ramjirv3217'
    }

    stages {
        stage("Checkout Code") {
            steps {
                checkout scm
            }
        }

        stage("Test Backend with Jest") {
            steps {
                dir("backend") {
                    sh '''
                    echo "Installing backend dependencies"
                    npm install

                    echo "Running backend tests"
                    npm run test
                    '''
                }
            }
        }

        stage("SonarQube Analysis - Backend") {
            steps {
                dir("backend") {
                    sh '''
                    echo "Running SonarQube analysis for backend"
                    sonar-scanner \
                        -Dsonar.projectKey=backend-first \
                        -Dsonar.host.url=$SONAR_HOST_URL \
                        -Dsonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage("Test Frontend and Build") {
            steps {
                dir("frontend") {
                    sh '''
                    echo "Installing frontend dependencies"
                    npm install

                    echo "Running frontend tests"
                    npm run test

                    echo "Building the frontend"
                    npm run build
                    '''
                }
            }
        }

        stage("SonarQube Analysis - Frontend") {
            steps {
                dir("frontend") {
                    sh '''
                    echo "Running SonarQube analysis for frontend"
                    sonar-scanner \
                        -Dsonar.projectKey=frontend-first \
                        -Dsonar.host.url=$SONAR_HOST_URL \
                        -Dsonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage("Build & Push Backend Docker Image") {
            steps {
                dir("backend") {
                    withCredentials([string(credentialsId: 'DOCKERHUB_PASS', variable: 'PASS')]) {
                        sh '''
                        echo "Building backend Docker image"
                        docker build -t backend-image .

                        echo "$PASS" | docker login -u $DOCKERHUB_USER --password-stdin
                        docker tag backend-image $DOCKERHUB_USER/backend-image:latest
                        docker push $DOCKERHUB_USER/backend-image:latest
                        '''
                    }
                }
            }
        }

        stage("Build & Push Frontend Docker Image") {
            steps {
                dir("frontend") {
                    withCredentials([string(credentialsId: 'DOCKERHUB_PASS', variable: 'PASS')]) {
                        sh '''
                        echo "Building frontend Docker image"
                        docker build -t frontend-image .

                        echo "$PASS" | docker login -u $DOCKERHUB_USER --password-stdin
                        docker tag frontend-image $DOCKERHUB_USER/frontend-image:latest
                        docker push $DOCKERHUB_USER/frontend-image:latest
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished"
        }
    }
}
