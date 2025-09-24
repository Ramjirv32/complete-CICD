pipeline {
    agent any

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
                          -Dsonar.host.url=http://localhost:9000 \
                          -Dsonar.token=sqa_71205c43f8c263e313ca3c193a82ed20faff52af
                    '''
                }
            }
        }

        stage("Build & Push Backend Docker Image") {
            steps {
                dir('backend') {
                    sh '''
                        docker build -t ramjirv3217/backend-image:latest .
                        docker login -u ramjirv3217 -p Kpr@23112005
                        docker push ramjirv3217/backend-image:latest
                    '''
                }
            }
        }

        stage("Build & Push Frontend Docker Image") {
            steps {
                dir('frontend') {
                    sh '''
                        docker build -t ramjirv3217/frontend-image:latest .
                        docker login -u ramjirv3217 -p Kpr@23112005
                        docker push ramjirv3217/frontend-image:latest
                    '''
                }
            }
        }

        stage("Deploy to Local Kubernetes") {
            steps {
                sh '''
                    echo "Pulling latest Docker images for local Kubernetes"
                    docker pull ramjirv3217/backend-image:latest
                    docker pull ramjirv3217/frontend-image:latest

                    echo "Applying Kubernetes manifests"
                    cd "/home/ramji/desktop/re/argo cd/k8"

                    export KUBECONFIG=/home/ramji/.kube/config

                    /usr/local/bin/minikube kubectl -- apply -f b.yaml
                    /usr/local/bin/minikube kubectl -- apply -f f.yaml

                    echo "Exposing backend and frontend services"
                    /usr/local/bin/minikube kubectl -- expose deployment backend --type=NodePort --port=5000 --dry-run=client -o yaml | /usr/local/bin/minikube kubectl -- apply -f -
                    /usr/local/bin/minikube kubectl -- expose deployment frontend --type=NodePort --port=5173 --dry-run=client -o yaml | /usr/local/bin/minikube kubectl -- apply -f -
                '''
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
