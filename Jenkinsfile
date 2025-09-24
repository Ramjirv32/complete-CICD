pipeline {
    agent any

    stages {
        stage('SCM Checkout') {
            steps {
                checkout scm
            }
        }

        stage("Backend local test") {
            steps {
                dir('backend') {
                    sh 'rm -rf package-lock.json'
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }

        stage('Building Frontend') {
            steps {
                dir('frontend') {
                    sh 'rm -rf package-lock.json'
                    sh 'npm install'
                    sh 'npm run build'
                }
            }
        }

        stage('SonarQube Analysis - Backend') {
            steps {
                script {
                    dir("backend") {
                        sh '''
                            export NVM_DIR="$HOME/.nvm"
                            if [ -s "$NVM_DIR/nvm.sh" ]; then
                                . "$NVM_DIR/nvm.sh"
                                nvm use 22
                            else
                                export PATH="/home/ramji/.nvm/versions/node/v22.19.0/bin:$PATH"
                            fi
                            which sonar-scanner
                            sonar-scanner --version
                            sonar-scanner \\
                              -Dsonar.projectKey=backend-first \\
                              -Dsonar.sources=. \\
                              -Dsonar.host.url=http://localhost:9000 \\
                              -Dsonar.token=sqa_71205c43f8c263e313ca3c193a82ed20faff52af
                        '''
                    }
                }
            }
        }

        stage("Building Docker for Backend and Pushing to DockerHub") {
            steps {
                dir('backend') {
                    sh 'docker build -t ramjirv3217/backend-image:latest .'
                    sh 'docker login -u "ramjirv3217" -p "Kpr@23112005"'
                    sh 'docker push ramjirv3217/backend-image:latest'
                }
            }
        }

        stage("Building Docker for Frontend and Pushing to DockerHub") {
            steps {
                dir('frontend') {
                    sh 'docker build -t ramjirv3217/frontend-image:latest .'
                    sh 'docker login -u "ramjirv3217" -p "Kpr@23112005"'
                    sh 'docker push ramjirv3217/frontend-image:latest'
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
