pipeline {
    agent any

    environment {
        SONAR_HOST_URL = 'https://lynx-fun-normally.ngrok-free.app'
        SONAR_TOKEN    = 'sqp_15da2dada419712d578bc42619572ae7f5168f03'
    }

    stages {
        stage("SonarQube Analysis - Backend") {
            steps {
                dir("backend") {
                    sh '''
                        echo "Running SonarQube analysis for backend"

                        export NVM_DIR="$HOME/.nvm"
                        [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
                        nvm use 22

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
            echo " Backend SonarQube analysis completed"
        }
        failure {
            echo "Backend SonarQube analysis failed"
        }
        success {
            echo "Backend SonarQube analysis succeeded"
        }
    }
}
