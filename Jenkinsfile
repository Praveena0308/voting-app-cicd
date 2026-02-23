pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_CREDENTIALS = credentials('docker-hub-credentials')
        GIT_COMMIT = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        IMAGE_TAG = "${BUILD_NUMBER}-${GIT_COMMIT.take(8)}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-credentials',
                    url: 'https://github.com/Praveena0308/voting-app-cicd.git'
            }
        }
        
        stage('Build Services') {
            parallel {
                // ===== VOTE SERVICE (Python) =====
                stage('Build Vote') {
                    agent {
                        docker {
                            image 'docker:25.0-dind'  // Use this instead of python:3.9-slim
                            reuseNode true
                            args '-u root:root -v /var/run/docker.sock:/var/run/docker.sock'
                        }
                    }
                    steps {
                        dir('vote') {
                             sh '''
                # Alpine uses python3-venv, not py3-venv
                apk add --no-cache python3 py3-pip python3-dev
                
                # Create virtual environment (Alpine way)
                python3 -m venv /venv
                source /venv/bin/activate
                
                pip install --no-cache-dir -r requirements.txt
                docker build -t vote-app:${IMAGE_TAG} .
            '''
                        }
                    }
                }
                
                // ===== RESULT SERVICE (Node.js) =====
                stage('Build Result') {
                    agent {
                        docker {
                            image 'docker:25.0-dind'
                            reuseNode true
                            args '-u root:root -v /var/run/docker.sock:/var/run/docker.sock'
                        }
                    }
                    steps {
                        dir('result') {
                            sh '''
                               apk add --no-cache nodejs npm  # Install Node.js
                               npm ci
                               docker build -t result-app:${IMAGE_TAG} .
          '''
                        }
                    }
                }
                
                // ===== WORKER SERVICE (.NET 7.0) =====
                stage('Build Worker') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/dotnet/sdk:7.0'  // Changed to 7.0
                            reuseNode true
                            args '-u root:root -v /var/run/docker.sock:/var/run/docker.sock'
                        }
                    }
                    steps {
                        dir('worker') {
                          sh '''
                # Install Docker CLI
                apt-get update && apt-get install -y docker.io
                
                # Set Docker API version
                export DOCKER_API_VERSION=1.44
                
                echo "📦 Restoring NuGet packages for linux-x64..."
                dotnet restore Worker.csproj -r linux-x64
                
                echo "🧹 Cleaning..."
                dotnet clean
                
                echo "🔨 Building for linux-x64..."
                dotnet build Worker.csproj -c Release -r linux-x64 --no-restore
                
                echo "📦 Publishing for linux-x64..."
                dotnet publish Worker.csproj -c Release -o /app -r linux-x64 --no-self-contained
                
                echo "🐳 Building Docker image..."
                docker build -t worker-app:${IMAGE_TAG} .
            '''
                        }
                    }
                }
            }
        }
        
        stage('Package & Push') {
            steps {
                script {
                    sh """
                        docker tag vote-app:${IMAGE_TAG} ${DOCKER_REGISTRY}/pravee033/vote-app:latest
                        docker tag result-app:${IMAGE_TAG} ${DOCKER_REGISTRY}/pravee033/result-app:latest
                        docker tag worker-app:${IMAGE_TAG} ${DOCKER_REGISTRY}/pravee033/worker-app:latest
                        docker tag vote-app:${IMAGE_TAG} ${DOCKER_REGISTRY}/pravee033/vote-app:${IMAGE_TAG}
                        docker tag result-app:${IMAGE_TAG} ${DOCKER_REGISTRY}/pravee033/result-app:${IMAGE_TAG}
                        docker tag worker-app:${IMAGE_TAG} ${DOCKER_REGISTRY}/pravee033/worker-app:${IMAGE_TAG}
                    """
                    
                    docker.withRegistry("https://index.docker.io/v1/", 'docker-hub-credentials') {
                        sh """
                            docker push ${DOCKER_REGISTRY}/pravee033/vote-app:${IMAGE_TAG}
                            docker push ${DOCKER_REGISTRY}/pravee033/result-app:${IMAGE_TAG}
                            docker push ${DOCKER_REGISTRY}/pravee033/worker-app:${IMAGE_TAG}
                            docker push ${DOCKER_REGISTRY}/pravee033/vote-app:latest
                            docker push ${DOCKER_REGISTRY}/pravee033/result-app:latest
                            docker push ${DOCKER_REGISTRY}/pravee033/worker-app:latest
                        
                        
    """
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo "🎉 Pipeline successful! Images pushed to Docker Hub!"
        }
        failure {
            echo "❌ Pipeline failed! Check logs for details."
        }
    }
}