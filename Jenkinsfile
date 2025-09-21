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
                    def scannerHome = tool 'SonarScanner'
                    dir("backend") {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \\
                              -Dsonar.projectKey=backend-first \\
                              -Dsonar.sources=. \\
                              -Dsonar.host.url=http://localhost:9000 \\
                              -Dsonar.login=sqp_15da2dada419712d578bc42619572ae7f5168f03
                        """
                    }
                }
            }
        }

        stage('Wait for Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    script {
                        def taskFile = readFile('backend/.scannerwork/report-task.txt')
                        def taskUrl = taskFile.split("\n").find { it.startsWith('ceTaskId=') }?.split('=')[1]
                        if (!taskUrl) {
                            error "Could not find SonarQube task ID!"
                        }

                        taskUrl = "http://localhost:9000/api/ce/task?id=${taskUrl}"
                        def status = "PENDING"
                        while (status != "SUCCESS" && status != "FAILED") {
                            sleep 5
                            def response = sh(script: "curl -s -u sqp_15da2dada419712d578bc42619572ae7f5168f03: '${taskUrl}'", returnStdout: true).trim()
                            status = readJSON(text: response).task.status
                        }

                        if (status == "FAILED") {
                            error "SonarQube analysis failed!"
                        }
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
