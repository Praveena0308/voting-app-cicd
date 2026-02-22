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
                // ===== VOTE SERVICE (Python/Flask) =====
                stage('Build Vote') {
                    agent {
                        docker {
                            image 'python:3.9-slim'
                            reuseNode true
                            args '-u root:root'
                        }
                    }
                    steps {
                        dir('vote') {
                            sh '''
                                echo "🐍 Building Vote Service (Python)..."
                                echo "Current directory: $(pwd)"
                                echo "Files: $(ls -la)"
                                
                                # Install Python dependencies
                                if [ -f "requirements.txt" ]; then
                                    echo "📦 Installing from requirements.txt..."
                                    pip install --no-cache-dir -r requirements.txt
                                else
                                    echo "⚠️ No requirements.txt found"
                                fi
                                
                                # Build Docker image
                                echo "🐳 Building Docker image..."
                                docker build -t vote-app:${IMAGE_TAG} .
                                docker tag vote-app:${IMAGE_TAG} vote-app:latest
                            '''
                        }
                    }
                }
                
                // ===== RESULT SERVICE (Node.js) =====
                stage('Build Result') {
                    agent {
                        docker {
                            image 'node:16-alpine'
                            reuseNode true
                            args '-u root:root'
                        }
                    }
                    steps {
                        dir('result') {
                            sh '''
                                echo "🟩 Building Result Service (Node.js)..."
                                echo "Current directory: $(pwd)"
                                echo "Files: $(ls -la)"
                                
                                # Set npm cache to writable location
                                mkdir -p /tmp/npm-cache
                                npm config set cache /tmp/npm-cache --global
                                
                                # Install Node dependencies
                                if [ -f "package.json" ]; then
                                    echo "📦 Running npm install..."
                                    npm ci || npm install
                                else
                                    echo "❌ package.json not found!"
                                    exit 1
                                fi
                                
                                # Build Docker image
                                echo "🐳 Building Docker image..."
                                docker build -t result-app:${IMAGE_TAG} .
                                docker tag result-app:${IMAGE_TAG} result-app:latest
                            '''
                        }
                    }
                }
                
                // ===== WORKER SERVICE (.NET) =====
                stage('Build Worker') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/dotnet/sdk:6.0'
                            reuseNode true
                            args '-u root:root'
                        }
                    }
                    steps {
                        dir('worker') {
                            sh '''
                                echo "🎯 Building Worker Service (.NET)..."
                                echo "Current directory: $(pwd)"
                                echo "Files: $(ls -la)"
                                
                                # Restore .NET dependencies
                                if [ -f "Worker.csproj" ]; then
                                    echo "📦 Restoring .NET packages..."
                                    dotnet restore Worker.csproj
                                    
                                    echo "🔨 Building .NET application..."
                                    dotnet build Worker.csproj -c Release
                                else
                                    echo "❌ Worker.csproj not found!"
                                    exit 1
                                fi
                                
                                # Build Docker image
                                echo "🐳 Building Docker image..."
                                docker build -t worker-app:${IMAGE_TAG} .
                                docker tag worker-app:${IMAGE_TAG} worker-app:latest
                            '''
                        }
                    }
                }
            }
        }
        
        stage('Package Docker Images') {
            steps {
                script {
                    sh """
                        echo "📦 Tagging images for registry..."
                        docker tag vote-app:${IMAGE_TAG} ${DOCKER_REGISTRY}/pravee033/vote-app:latest
                        docker tag result-app:${IMAGE_TAG} ${DOCKER_REGISTRY}/pravee033/result-app:latest
                        docker tag worker-app:${IMAGE_TAG} ${DOCKER_REGISTRY}/pravee033/worker-app:latest
                        
                        docker tag vote-app:${IMAGE_TAG} ${DOCKER_REGISTRY}/pravee033/vote-app:${IMAGE_TAG}
                        docker tag result-app:${IMAGE_TAG} ${DOCKER_REGISTRY}/pravee033/result-app:${IMAGE_TAG}
                        docker tag worker-app:${IMAGE_TAG} ${DOCKER_REGISTRY}/pravee033/worker-app:${IMAGE_TAG}
                    """
                }
            }
        }
        
        stage('Push to Registry') {
            steps {
                script {
                    docker.withRegistry("https://index.docker.io/v1/", 'docker-hub-credentials') {
                        sh """
                            echo "📤 Pushing images to Docker Hub..."
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
            echo "🎉 Pipeline successful! Images pushed:"
            echo "  - ${DOCKER_REGISTRY}/pravee033/vote-app:${IMAGE_TAG}"
            echo "  - ${DOCKER_REGISTRY}/pravee033/result-app:${IMAGE_TAG}"
            echo "  - ${DOCKER_REGISTRY}/pravee033/worker-app:${IMAGE_TAG}"
        }
        failure {
            echo "❌ Pipeline failed! Check logs for details."
        }
    }
}