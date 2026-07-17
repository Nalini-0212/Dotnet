pipeline {
    agent any

    tools {
        jdk 'JDK17'
    }

    environment {
        SCANNER_HOME = tool('sonar-scanner')
        IMAGE = 'naliniselv/donet:latest'
        AWS_REGION = 'ap-south-1'
        CLUSTER_NAME = 'dotnet-cluster'
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout From Git') {
            steps {
                git(
                    branch: 'main',
                    credentialsId: 'github-creds',
                    url: 'https://github.com/Nalini-0212/Dotnet.git'
                )
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

        stage('Docker Build & Tag') {
            steps {
                sh """
                docker build -t ${IMAGE} -f build/Dockerfile .
                """
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh """
                trivy image ${IMAGE} > trivy-image-report.txt
                """
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhubcred',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push ${IMAGE}
                    docker logout
                    '''
                }
            }
        }

        stage('Configure EKS') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                     credentialsId: 'aws-creds']
                ]) {

                    echo "========== CONFIGURE EKS =========="

                    sh """
                    aws sts get-caller-identity

                    aws eks update-kubeconfig \
                        --region ${AWS_REGION} \
                        --name ${CLUSTER_NAME}
                    """
                }
            }
        }

        stage('Verify EKS Cluster') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                     credentialsId: 'aws-creds']
                ]) {

                    echo "========== VERIFY EKS =========="

                    sh '''
                    kubectl get nodes
                    kubectl get namespaces
                    '''
                }
            }
        }

        stage('Deploy Kubernetes Manifests') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                     credentialsId: 'aws-creds']
                ]) {

                    echo "========== APPLY MANIFESTS =========="

                    sh '''
                    kubectl apply -f k8s/deployment.yml
                    kubectl apply -f k8s/service.yml
                    '''
                }
            }
        }

        stage('Deploy Latest Image') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                     credentialsId: 'aws-creds']
                ]) {

                    echo "========== UPDATE DEPLOYMENT IMAGE =========="

                    sh """
                    kubectl set image deployment/dotnet-app-deployment \
                    dotnet-app=${IMAGE}
                    """
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                     credentialsId: 'aws-creds']
                ]) {

                    echo "========== VERIFY DEPLOYMENT =========="

                    sh """
                    kubectl rollout status deployment/dotnet-app --timeout=5m

                    kubectl get deployments

                    kubectl get pods

                    kubectl get svc

                    kubectl describe deployment dotnet-app
                    """
                }
            }
        }

        stage('Get LoadBalancer Details') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                     credentialsId: 'aws-creds']
                ]) {

                    echo "========== LOAD BALANCER DETAILS =========="

                    sh """
                    sleep 60

                    kubectl get svc dotnet-service -o wide

                    kubectl describe svc dotnet-service
                    """
                }
            }
        }
    }

    post {

        always {
            echo "========== CLEANUP =========="

            archiveArtifacts(
                artifacts: 'trivy-fs_report.txt,trivy-image-report.txt',
                allowEmptyArchive: true
            )
        }

        success {
            echo 'Pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed. Check Jenkins logs.'
        }
    }
}