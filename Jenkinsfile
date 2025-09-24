pipeline {
    agent any

    stages {
        stage('SCM Checkout') {
            steps {
                checkout scm
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
                            
                            # Run the analysis
                            sonar-scanner \\
                              -Dsonar.projectKey=backend-first \\
                              -Dsonar.sources=. \\
                              -Dsonar.host.url=http://localhost:9000 \\
                              -Dsonar.token=sqp_15da2dada419712d578bc42619572ae7f5168f03
                        '''
                    }
                }
            }
        }

        stage('Wait for Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    script {
                        def taskFile = readFile('backend/.scannerwork/report-task.txt')
                        def taskId = taskFile.split("\\n").find { it.startsWith('ceTaskId=') }?.split('=')[1]
                        if (!taskId) {
                            error "Could not find SonarQube task ID in report-task.txt"
                        }

                        def taskUrl = "http://localhost:9000/api/ce/task?id=${taskId}"
                        def status = "PENDING"
                        echo "Checking task status at: ${taskUrl}"
                        
                        while (status != "SUCCESS" && status != "FAILED") {
                            sleep 5
                            def response = sh(
                                script: "curl -s -u sqp_15da2dada419712d578bc42619572ae7f5168f03: '${taskUrl}'", 
                                returnStdout: true
                            ).trim()
                            
                            def jsonResponse = readJSON(text: response)
                            status = jsonResponse.task.status
                            echo "Current task status: ${status}"
                        }

                        if (status == "FAILED") {
                            error "SonarQube Quality Gate failed!"
                        }
                        
                        echo "SonarQube analysis completed successfully"
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Backend SonarQube stage completed"
        }
        success {
            echo "Backend SonarQube analysis and Quality Gate succeeded"
        }
        failure {
            echo "Backend SonarQube analysis failed"
        }
    }
}