pipeline {
    agent any

    tools {
        maven 'maven'
    }

    environment {
        IMAGE_NAME = "mydocker3692/netflix-clone"
        TAG = "${BUILD_NUMBER}"
        MAVEN_HOME = tool 'maven'
        SCANNER_HOME = tool 'sonar-scanner'
        KUBECONFIG = "/var/lib/jenkins/.kube/config"
        TRIVY_CACHE_DIR = "/var/lib/jenkins/trivycache"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git 'https://github.com/yesk993-ops/Netflix-clone.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONAR_SERVER}") {
                    sh '''
                    sonar-scanner \
                    -Dsonar.projectKey=netflix-clone \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=http://localhost:9000 \
                    -Dsonar.login=$SONAR_AUTH_TOKEN
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$TAG .'
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh '''
                trivy image $IMAGE_NAME:$TAG
                '''
            }
        }

        stage('Push Image') {
            steps {
                withDockerRegistry(credentialsId: 'docker', url: '') {
                    sh 'docker push $IMAGE_NAME:$TAG'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                '''
            }
        }

    }
}
