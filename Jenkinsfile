pipeline {
    // Only ONE agent declaration at top level!
    agent any  // This runs on Jenkins agent, then each stage can override
    
    triggers {
        // This enables GitHub webhook trigger
        githubPush()
        
    environment {
        // Define your variables!
        DOCKER_REGISTRY = 'docker.io'  // or your registry
        DOCKER_CREDENTIALS = credentials('docker-hub-credentials')
        IMAGE_TAG = "${BUILD_NUMBER}-${GIT_COMMIT[0..7]}"  // Unique tag
        GIT_COMMIT = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
    }
    
    stages {
        stage('Checkout') {
            steps {
                // Fixed: Added quotes around URL
                git branch: 'main',
                    credentialsId: 'github-credentials',
                    url: 'https://github.com/Praveena0308/voting-app-cicd.git'
            }
        }
        
        stage('Build Services') {
            parallel {
                stage('Build Vote') {
                    agent {
                        docker {
                            image 'node:14'
                            reuseNode true  // Reuse workspace from checkout
                        }
                    }
                    steps {
                        dir('vote') {  // Added missing braces!
                            sh '''
                                echo "Building Vote app..."
                                npm install
                                npm run build
                                docker build -t vote-app:${IMAGE_TAG} .
                            '''
                        }
                    }
                }
                
                stage('Build Result') {
                    agent {
                        docker {
                            image 'node:16-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        dir('result') {  // Added missing braces!
                            sh '''
                                echo "Building Result app..."
                                npm install
                                npm run build
                                docker build -t result-app:${IMAGE_TAG} .
                            '''
                        }
                    }
                }
                
                stage('Build Worker') {
                    agent {
                        docker {
                            image 'python:3.9-slim'
                            reuseNode true
                        }
                    }
                    steps {
                        dir('worker') {  // Added missing braces!
                            sh '''
                                echo "Building Worker app..."
                                pip install -r requirements.txt
                                docker build -t worker-app:${IMAGE_TAG} .
                            '''
                        }
                    }
                }
            }  // Close parallel
        }  // Close Build Services stage
        
        stage('Package Docker Images') {
            steps {
                script {
                    sh """
                        echo "Tagging images for registry..."
                        docker tag vote-app:${IMAGE_TAG} ${DOCKER_REGISTRY}/pravee033/vote-app:latest
                        docker tag result-app:${IMAGE_TAG} ${DOCKER_REGISTRY}/pravee033/result-app:latest
                        docker tag worker-app:${IMAGE_TAG} ${DOCKER_REGISTRY}/pravee033/worker-app:latest
                        
                        # Also tag with build number for versioning
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
                            echo "Pushing images to Docker Hub..."
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
    }  // Close stages
    
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