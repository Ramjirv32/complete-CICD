pipeline {
    agent any

    environment {
        DOCKER_USER = "ramjirv3217"
        DOCKER_PASS = "Kpr@23112005"
        SONAR_TOKEN = "sqa_71205c43f8c263e313ca3c193a82ed20faff52af"
        SONAR_HOST = "http://localhost:9000"
        K8S_MANIFEST_DIR = "/home/ramji/desktop/re/argo cd/k8"
    }

    stages {
        stage('SCM Checkout') {
            steps {
                checkout scm
            }
        }

// d
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
                    sh '''
                        docker build -t ${DOCKER_USER}/backend-image:latest .
                        docker login -u ${DOCKER_USER} -p ${DOCKER_PASS}
                        docker push ${DOCKER_USER}/backend-image:latest
                    '''
                }
            }
        }

        stage("Build & Push Frontend Docker Image") {
            steps {
                dir('frontend') {
                    sh '''
                        docker build -t ${DOCKER_USER}/frontend-image:latest .
                        docker login -u ${DOCKER_USER} -p ${DOCKER_PASS}
                        docker push ${DOCKER_USER}/frontend-image:latest
                    '''
                }
            }
        }

// k/
  stage("Deploy to Local Kubernetes") {
stage("Deploy to Local Kubernetes") {
    steps {
        sh '''
            echo "Pulling latest Docker images for local Kubernetes"
            docker pull ramjirv3217/backend-image:latest
            docker pull ramjirv3217/frontend-image:latest

            echo "Applying Kubernetes manifests"
            cd "/home/ramji/desktop/re/argo cd/k8"

            /usr/local/bin/minikube kubectl -- apply -f b.yaml
            /usr/local/bin/minikube kubectl -- apply -f f.yaml

            echo "Exposing backend and frontend services"
            /usr/local/bin/minikube kubectl -- expose deployment backend --type=NodePort --port=5000 --dry-run=client -o yaml | /usr/local/bin/minikube kubectl -- apply -f -
            /usr/local/bin/minikube kubectl -- expose deployment frontend --type=NodePort --port=5173 --dry-run=client -o yaml | /usr/local/bin/minikube kubectl -- apply -f -
        '''
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
