pipeline {
    agent any
    
    environment {
        // Defines global variables
        DOCKER_IMAGE = "eswar199918/devops-project"
        REGISTRY_CRED = "docker-cred" 
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }
        
        stage('Code Quality (SonarQube)') {
            steps {
                script {
                    // Uses the scanner tool we installed
                    def scannerHome = tool 'SonarScanner' 
                    withSonarQubeEnv('sonar-server') {
                        sh "${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=devops-project \
                        -Dsonar.sources=."
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build(DOCKER_IMAGE + ":${env.BUILD_NUMBER}")
                }
            }
        }

        stage('Security Scan (Trivy)') {
            steps {
                sh "trivy image --severity HIGH,CRITICAL ${DOCKER_IMAGE}:${env.BUILD_NUMBER}"
                // Note: In real life, you might want to 'exit 1' here if critical bugs are found.
                // For now, we just log it so the build doesn't fail while learning.
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('', REGISTRY_CRED) {
                        dockerImage.push()
                        dockerImage.push('latest')
                    }
                }
            }
        }
    }
}
