pipeline {
    agent any

    stages {
        stage('SCM Checkout') {
            steps {
                checkout scm
            }
        }

    }
    stage("Backend local test"){
        steps{
            dir('backend'){
                sh 'rm -rf package-lock.json'
                sh 'npm install'
                sh 'npm test'
            }
        }
    }

    stage('Building Frontend'){
        steps{
            dir('frontend'){
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
                            # Load NVM and set up environment
                            export NVM_DIR="$HOME/.nvm"
                            if [ -s "$NVM_DIR/nvm.sh" ]; then
                                . "$NVM_DIR/nvm.sh"
                                nvm use 22
                            else
                                # Fallback: directly add the path where sonar-scanner exists
                                export PATH="/home/ramji/.nvm/versions/node/v22.19.0/bin:$PATH"
                            fi
                            
                            # Verify sonar-scanner is available
                            which sonar-scanner
                            sonar-scanner --version
                            
                            # Run the analysis with the new working token
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


stage("building docker for backend and pushing to dockerhub"){
    steps{
        dir('backend'){
            sh 'docker build -t ramjirv3217/backend-image:latest .'
            sh 'docker login -u "ramjirv3217" -p "Kpr@23112005"'
            sh 'docker push ramjirv3217/backend-image:latest'
        }


    }

}

stage("Building docker for frontend and pushing to dockerhub"){
    steps{
        dir('frontend'){
            sh 'docker build -t ramjirv3217/frontend-image:latest .'
            sh 'docker login -u "ramjirv3217" -p "Kpr@23112005"'
            sh 'docker push ramjirv3217/frontend-image:latest'
        }
    }


}


post {
        always {
            cleanWs()
        }
    }


}
