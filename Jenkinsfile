pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        SCANNER_HOME = tool 'SonarQube Scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/shantanudatarkar/Ekart.git'
            }
        }

        stage('COMPILE') {
            steps {
                sh "mvn clean compile -DskipTests=true"
            }
        }

        stage('Static Code Analysis') {
            environment {
                SONAR_URL = "http://13.233.148.124:9000/"
            }
            steps {
                withCredentials([string(credentialsId: 'sonarqube', variable: 'sonarqube')]) {
                    echo "SonarQube Token: ${sonarqube}"
                    sh 'mvn sonar:sonar -Dsonar.login=${sonarqube} -Dsonar.host.url=${SONAR_URL}'
                }
            }
        }

        stage('Build') {
            steps {
                sh "mvn clean package -DskipTests=true"
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/') {
                        def buildNumber = env.BUILD_NUMBER ?: 'latest'
                        sh "docker build -t shan123456/docker_demo -f docker/Dockerfile ."
                        sh "docker tag shan123456/docker_demo:latest shan123456/docker_demo:${buildNumber}"
                        sh "docker push shan123456/docker_demo:${buildNumber}"
                        sh "docker push shan123456/docker_demo:latest"
                    }
                }
            }
        }

        stage('Kubernetes Deploy') {
            steps {
                script {
                    // Clone the Helm chart repository
                    sh "rm -rf *"
                    git branch: 'master', url: 'https://github.com/shantanu-da/deployment.git'

                    kubeconfig(credentialsId: 'kubernetes', serverUrl: '') {
                        // Delete all pods before deploying (optional)
                        sh  "ls"
                        sh  "pwd"
                        sh "kubectl get pods"

                        // Install the Helm chart for deployment
                        sh """
                           cd /var/lib/jenkins/workspace/Ekart/shopping-cart
                            helm upgrade --install shopping-cart .

                           """
                    }
                }
            }
        }
    }
}
