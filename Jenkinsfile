pipeline {
    agent any

    tools {
        jdk 'JDK17'
    }

    environment {
        SCANNER_HOME = tool('sonar-scanner')
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout From Git') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Nalini-0212/Dotnet.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh """
                        ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectName=Dotnet-Webapp \
                        -Dsonar.projectKey=Dotnet-Webapp
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        stage('Trivy File Scan') {
            steps {
                sh 'trivy fs . > trivy-fs_report.txt'
            }
        }

        stage('OWASP Dependency Check') {
            steps {

                withCredentials([
                    string(credentialsId: 'nvd-api-key', variable: 'APIKEY')
                ]) {

                    dependencyCheck(
                        odcInstallation: 'DP-Check',
                        additionalArguments: """
                            --scan .
                            --format XML
                            --nvdApiKey $APIKEY
                        """
                    )

                    dependencyCheckPublisher(
                        pattern: '**/dependency-check-report.xml'
                    )
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts(
                artifacts: 'trivy-fs_report.txt',
                allowEmptyArchive: true
            )
        }

        success {
            echo 'Pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed. Check stage logs.'
        }
    }
}