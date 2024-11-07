pipeline {
    agent any

    tools {
        nodejs "Nodejs-node21"
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        TRIVY_CACHE_DIR = '/home/ubuntu/.cache/trivy'
        TRIVY_DB_DIR = '/home/ubuntu/.cache/trivy/db'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Bharathkj12/3-Tier-Full-Stack.git'
            }
        }

        stage('Install Package Dependencies') {
            steps {
                sh "npm install"
            }
        }

        stage('Run Test Cases') {
            steps {
                sh "npm test"
            }
        }

        stage('Trivy FS Scan') {
            steps {
                script {
                    try {
                        sh "trivy fs --format table -o fs-report.html ."
                    } catch (Exception e) {
                        echo "Trivy FS Scan failed: ${e.getMessage()}"
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=Campground -Dsonar.projectName=Campground"
                }
            }
        }

        stage('Docker Build and Tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'Docker paswd', toolName: 'docker') {
                        sh "docker build -t kjbharath/campa:latest ."
                    }
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                script {
                    try {
                        sh "trivy image --format table -o image-scan-report.html kjbharath/camp:latest"
                    } catch (Exception e) {
                        echo "Trivy Image Scan failed: ${e.getMessage()}"
                    }
                }
            }
        }

        stage('Docker Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'Docker paswd', toolName: 'docker') {
                        sh "docker push kjbharath/campa:latest"
                    }
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                script {
                    try {
                        withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'my-eks22', contextName: '', credentialsId: 'k8s-token', namespace: 'webapps', serverUrl: 'https://F79008A01C4DEFA4C74D5D0A9D73BE1B.gr7.ap-south-1.eks.amazonaws.com']]) {
                            sh "kubectl apply -f Manifests/dss.yml"
                            sleep 60
                        }
                    } catch (Exception e) {
                        echo "Deployment to EKS failed: ${e.getMessage()}"
                    }
                }
            }
        }

        stage('Verify the Deployment') {
            steps {
                script {
                    try {
                        withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'my-eks22', contextName: '', credentialsId: 'k8s-token', namespace: 'webapps', serverUrl: 'https://F79008A01C4DEFA4C74D5D0A9D73BE1B.gr7.ap-south-1.eks.amazonaws.com']]) {
                            sh "kubectl get pods -n webapps"
                            sh "kubectl get svc -n webapps"
                        }
                    } catch (Exception e) {
                        echo "Deployment verification failed: ${e.getMessage()}"
                    }
                }
            }
        }
    }
}
