pipeline {
    agent any

    environment {
        DOCKER_USER = "ramjirv3217"
        DOCKER_PASS = "Kpr@23112005"
        SONAR_TOKEN = "sqa_71205c43f8c263e313ca3c193a82ed20faff52af"
        SONAR_HOST = "http://localhost:9000"
    }

    stages {
        stage('SCM Checkout') {
            steps {
                checkout scm
            }
        }

        stage("Backend Tests") {
            steps {
                dir('backend') {
                    sh 'rm -rf package-lock.json'
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }

        stage('Frontend Build') {
            steps {
                dir('frontend') {
                    sh 'rm -rf package-lock.json'
                    sh 'npm install'
                    sh 'npm run build'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                dir('backend') {
                    withEnv(["SONAR_TOKEN=${SONAR_TOKEN}"]) {
                        sh '''
                            export NVM_DIR="$HOME/.nvm"
                            if [ -s "$NVM_DIR/nvm.sh" ]; then
                                . "$NVM_DIR/nvm.sh"
                                nvm use 22
                            else
                                export PATH="/home/ramji/.nvm/versions/node/v22.19.0/bin:$PATH"
                            fi

                            sonar-scanner \
                              -Dsonar.projectKey=backend-first \
                              -Dsonar.sources=. \
                              -Dsonar.host.url=${SONAR_HOST} \
                              -Dsonar.token=${SONAR_TOKEN}
                        '''
                    }
                }
            }
        }

        stage("Build & Push Backend Docker Image") {
            steps {
                dir('backend') {
                    sh 'docker build -t ${DOCKER_USER}/backend-image:latest .'
                    sh 'docker login -u ${DOCKER_USER} -p ${DOCKER_PASS}'
                    sh 'docker push ${DOCKER_USER}/backend-image:latest'
                }
            }
        }

        stage("Build & Push Frontend Docker Image") {
            steps {
                dir('frontend') {
                    sh 'docker build -t ${DOCKER_USER}/frontend-image:latest .'
                    sh 'docker login -u ${DOCKER_USER} -p ${DOCKER_PASS}'
                    sh 'docker push ${DOCKER_USER}/frontend-image:latest'
                }
            }
        }

        stage("Deploy to Kubernetes") {
            steps {
                dir('k8s') {
                    sh 'kubectl apply -f b.yaml'
                    sh 'kubectl apply -f f.yaml'
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
