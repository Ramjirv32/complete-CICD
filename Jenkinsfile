pipeline {
    agent any

    stages {
        stage("Test backend using Jest") {
            steps {
                checkout scm
                dir("backend") {
                    sh '''
                    echo "Removing package-lock.json"
                    rm -rf package-lock.json

                    echo "Installing dependencies"
                    npm install

                    echo "Running tests"
                    npm run test
                    '''
                }
            }
        }

        stage("SonarQube Analysis Backend") {
            steps {
                dir("backend") {
                    sh '''
                    echo "Running SonarQube analysis for backend"
                    sonar-scanner \
                        -Dsonar.projectKey=backend \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://your-sonarqube-server:9000 \
                        -Dsonar.login=your_sonarqube_token
                    '''
                }
            }
        }

        stage("Test frontend and build") {
            steps {
                dir("frontend") {
                    sh '''
                    echo "Removing package-lock.json"
                    rm -rf package-lock.json

                    echo "Installing dependencies"
                    npm install

                    echo "Running tests"
                    npm run test

                    echo "Building the frontend"
                    npm run build
                    '''
                }
            }
        }

        stage("SonarQube Analysis Frontend") {
            steps {
                dir("frontend") {
                    sh '''
                    echo "Running SonarQube analysis for frontend"
                    sonar-scanner \
                        -Dsonar.projectKey=frontend \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://your-sonarqube-server:9000 \
                        -Dsonar.login=your_sonarqube_token
                    '''
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
