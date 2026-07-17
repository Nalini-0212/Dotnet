pipeline {
    agent any

    tools {
        jdk 'JDK17'
    }

    environment {
        SCANNER_HOME = tool('sonar-scanner')
        NVD_API_KEY = credentials('nvd-api-key')
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
                dependencyCheck(
                    additionalArguments: "--scan ./ --format XML --nvdApiKey ${NVD_API_KEY}",
                    odcInstallation: 'DP-Check'
                )

                dependencyCheckPublisher(
                    pattern: '**/dependency-check-report.xml'
                )
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'trivy-fs_report.txt', allowEmptyArchive: true
        }
    }
}