pipeline {
    agent any

    environment {
        SONAR_HOST_URL  = 'https://lynx-fun-normally.ngrok-free.app'
        SONAR_TOKEN     = 'sqp_15da2dada419712d578bc42619572ae7f5168f03'
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
    }

    post {
        always {
            echo "Backend SonarQube pipeline finished"
        }
    }
}
