# Example Voting App

A simple distributed application running across multiple Docker containers.

## Getting started

Download [Docker Desktop](https://www.docker.com/products/docker-desktop) for Mac or Windows. [Docker Compose](https://docs.docker.com/compose) will be automatically installed. On Linux, make sure you have the latest version of [Compose](https://docs.docker.com/compose/install/).

This solution uses Python, Node.js, .NET, with Redis for messaging and Postgres for storage.

Run in this directory to build and run the app:

```shell
docker compose up
```

The `vote` app will be running at [http://localhost:8080](http://localhost:8080), and the `results` will be at [http://localhost:8081](http://localhost:8081).

Alternately, if you want to run it on a [Docker Swarm](https://docs.docker.com/engine/swarm/), first make sure you have a swarm. If you don't, run:

```shell
docker swarm init
```

Once you have your swarm, in this directory run:

```shell
docker stack deploy --compose-file docker-stack.yml vote
```

## Run the app in Kubernetes

The folder k8s-specifications contains the YAML specifications of the Voting App's services.

Run the following command to create the deployments and services. Note it will create these resources in your current namespace (`default` if you haven't changed it.)

```shell
kubectl create -f k8s-specifications/
```

The `vote` web app is then available on port 31000 on each host of the cluster, the `result` web app is available on port 31001.

To remove them, run:

```shell
kubectl delete -f k8s-specifications/
```

## Architecture

![Architecture diagram](architecture.excalidraw.png)

* A front-end web app in [Python](/vote) which lets you vote between two options
* A [Redis](https://hub.docker.com/_/redis/) which collects new votes
* A [.NET](/worker/) worker which consumes votes and stores them in…
* A [Postgres](https://hub.docker.com/_/postgres/) database backed by a Docker volume
* A [Node.js](/result) web app which shows the results of the voting in real time

## Notes

The voting application only accepts one vote per client browser. It does not register additional votes if a vote has already been submitted from a client.

This isn't an example of a properly architected perfectly designed distributed app... it's just a simple
example of the various types of pieces and languages you might see (queues, persistent data, etc), and how to
deal with them in Docker at a basic level.





From Zero to Hero: Your Complete Journey 🚀
<div align="center"> <img src="https://www.jenkins.io/images/jenkins-logo.png" alt="Jenkins Logo" width="200"/> <img src="https://www.docker.com/wp-content/uploads/2022/03/horizontal-logo-monochromatic-white.png" alt="Docker Logo" width="200"/> <img src="https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png" alt="GitHub Logo" width="100"/> </div><p align="center"> <strong>A comprehensive guide to setting up a complete CI/CD pipeline for microservices using Jenkins, Docker, and GitHub Webhooks</strong> </p>
📑 TABLE OF CONTENTS
📚 Complete Jenkins CI/CD Pipeline Documentation

From Zero to Hero: Your Complete Journey 🚀

📑 TABLE OF CONTENTS

1️⃣ JENKINS INSTALLATION & INITIAL SETUP

1.1 Install Jenkins on Ubuntu/Debian

1.2 Initial Jenkins Access

1.3 Install Docker (Required for Pipeline)

1.4 Essential Jenkins Plugins

2️⃣ JENKINS CONFIGURATION AS CODE (JCASC)

2.1 Understanding JCasC

2.2 Switch to Jenkins User

2.3 Create JCasC Configuration Directory

2.4 Create Jenkins.yaml Configuration File

2.5 Set Proper File Permissions

2.6 Configure Jenkins Service for JCasC

Method A: Using /etc/default/jenkins (Debian/Ubuntu)

Method B: Using systemd service file (More Reliable)

2.7 Restart Jenkins to Apply Changes

2.8 Verify JCasC Installation

3️⃣ GITHUB & DOCKER HUB CREDENTIALS SETUP

3.1 Generate GitHub Personal Access Token

3.2 Generate Docker Hub Access Token

3.3 Set Environment Variables

3.4 Verify Credentials in Jenkins

4️⃣ SMEE.IO WEBHOOK CONFIGURATION

4.1 Why Smee.io?

4.2 Install Smee Client

4.3 Create a Smee Channel

4.4 Start Smee Client

4.5 Configure GitHub Webhook

4.6 Test Webhook

5️⃣ JENKINS PIPELINE DEVELOPMENT

5.1 Create Pipeline Job

5.2 Understanding Jenkinsfile Structure

6️⃣ THE VOTING APP JENKINSFILE

Complete Final Jenkinsfile

7️⃣ TROUBLESHOOTING & COMMON ISSUES

Issue 1: Jenkins can't find Docker

Issue 2: "docker: not found" inside containers

Issue 3: Docker API version mismatch

Issue 4: Python externally-managed-environment error

Issue 5: .NET runtime identifier issues

Issue 6: JCasC not applying

Issue 7: Webhooks not triggering

8️⃣ KEY COMMANDS CHEAT SHEET

Jenkins Service Commands

JCasC Commands

Smee.io Commands

Docker Commands

Git Commands

Jenkins CLI Commands

🎓 LEARNING SUMMARY

What You've Mastered:

Your Pipeline Achievements:

📝 FINAL NOTES

1️⃣ JENKINS INSTALLATION & INITIAL SETUP
1.1 Install Jenkins on Ubuntu/Debian
bash
# Update system
sudo apt update
sudo apt upgrade -y

# Install Java (Jenkins requirement)
sudo apt install openjdk-11-jdk -y

java -version

# Add Jenkins repository
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

# Install Jenkins
sudo apt update
sudo apt install jenkins -y

# Start Jenkins
sudo systemctl start jenkins

sudo systemctl enable jenkins

sudo systemctl status jenkins

## 1.2 Initial Jenkins Access
bash
# Get initial admin password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
# Copy this password for first login

# Jenkins will be available at: http://localhost:8080
1.3 Install Docker (Required for Pipeline)
bash
# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add Jenkins user to docker group
sudo usermod -aG docker jenkins

# Verify
groups jenkins
docker --version

# Restart Jenkins to apply changes
sudo systemctl restart jenkins

## 1.4 Essential Jenkins Plugins
Install these plugins via Jenkins UI:

Pipeline (already installed)

Docker Pipeline - For Docker builds

GitHub Integration - For webhooks

Configuration as Code - For JCasC

Credentials Binding - For secure credentials

2️⃣ JENKINS CONFIGURATION AS CODE (JCASC)

## 2.1 Understanding JCasC
JCasC allows you to define Jenkins configuration in YAML files, making your setup reproducible and version-controlled.

2.3 Create JCasC Configuration Directory
bash


# Create casc_configs directory
sudo mkdir -p /var/lib/jenkins/casc_configs

# Set proper ownership
sudo chown -R jenkins:jenkins /var/lib/jenkins/casc_configs
2.4 Create Jenkins.yaml Configuration File
bash
# Create and edit the JCasC file
 

Paste this configuration:

yaml
# Jenkins Configuration as Code (JCasC)
jenkins:
  systemMessage: "Jenkins configured automatically by JCasC - Voting App Pipeline"
  numExecutors: 2
  mode: NORMAL
  scmCheckoutRetryCount: 3
  
  # Security settings
  securityRealm:
    local:
      allowsSignup: false
      users:
        - id: "admin"
          password: "${JENKINS_ADMIN_PASSWORD}"
  
  # Authorization strategy
  authorizationStrategy:
    loggedInUsersCanDoAnything:
      allowAnonymousRead: false

# Credentials configuration
    
    credentials:
    system:
    domainCredentials:
      - credentials:
          # GitHub credentials
          - usernamePassword:
              scope: GLOBAL
              id: "github-credentials"
              username: "${GITHUB_USERNAME}"
              password: "${GITHUB_TOKEN}"
              description: "GitHub credentials for checkout"
          
          # Docker Hub credentials  
          - usernamePassword:
              scope: GLOBAL
              id: "docker-hub-credentials"
              username: "${DOCKER_USERNAME}"
              password: "${DOCKER_TOKEN}"
              description: "Docker Hub for pushing images"

# Security headers and CSRF protection
unclassified:
  location:
    url: "http://localhost:8080/"
    adminAddress: "admin@example.com"
  
  # Global libraries (optional)
  globalLibraries:
    libraries:
      - name: "pipeline-utils"
        defaultVersion: "main"
        implicit: false
        retriever:
          modernSCM:
            scm:
              git:
                remote: "https://github.com/your-org/pipeline-utils.git"


2.5 Set Proper File Permissions
bash
# Secure the configuration file (read-only for jenkins user)
sudo chmod 600 /var/lib/jenkins/casc_configs/jenkins.yaml

# Verify permissions
ls -la /var/lib/jenkins/casc_configs/

2.6 Configure Jenkins Service for JCasC
There are two ways to configure JCasC:

Method A: Using /etc/default/jenkins (Debian/Ubuntu)
bash
# Edit Jenkins default configuration
sudo nano /etc/default/jenkins
Add these lines:

bash
# Java options - ADD THIS LINE
JAVA_ARGS="-Djava.awt.headless=true -Dcasc.jenkins.config=/var/lib/jenkins/casc_configs/jenkins.yaml"

# Environment variables - ADD THESE LINES at the end
export CASC_JENKINS_CONFIG=/var/lib/jenkins/casc_configs/jenkins.yaml
export GITHUB_USERNAME="Praveena0308"
export GITHUB_TOKEN="ghp_your_github_token_here"
export DOCKER_USERNAME="pravee033"
export DOCKER_TOKEN="dckr_pat_your_docker_token_here"


Method B: Using systemd service file (More Reliable)
bash
# Create backup first
sudo cp /lib/systemd/system/jenkins.service /lib/systemd/system/jenkins.service.backup

# Edit the service file
sudo nano /lib/systemd/system/jenkins.service
Find the [Service] section and modify/add:

ini
[Service]
# Add Java options for JCasC
Environment="JAVA_OPTS=-Djava.awt.headless=true -Dcasc.jenkins.config=/var/lib/jenkins/casc_configs/jenkins.yaml"

# Add environment variables for credentials
Environment="CASC_JENKINS_CONFIG=/var/lib/jenkins/casc_configs/jenkins.yaml"
Environment="GITHUB_USERNAME=Praveena0308"
Environment="GITHUB_TOKEN=ghp_your_github_token_here"
Environment="DOCKER_USERNAME=pravee033"
Environment="DOCKER_TOKEN=dckr_pat_your_docker_token_here"
2.7 Restart Jenkins to Apply Changes
bash
# Reload systemd configuration
sudo systemctl daemon-reload

# Restart Jenkins
sudo systemctl restart jenkins

# Check status
sudo systemctl status jenkins

# Check logs for errors
sudo journalctl -u jenkins -f
2.8 Verify JCasC Installation
bash
# Check if JCasC plugin is installed
ls -la /var/lib/jenkins/plugins/ | grep configuration-as-code

# Check if configuration was applied
# Visit Jenkins UI: Manage Jenkins → Configuration as Code
# Or check logs:
sudo journalctl -u jenkins | grep "Configuration as Code"


3️⃣ GITHUB & DOCKER HUB CREDENTIALS SETUP

3.1 Generate GitHub Personal Access Token
Go to GitHub.com → Settings → Developer settings → Personal access tokens → Tokens (classic)

Click "Generate new token (classic)"

Set scopes: repo, admin:repo_hook

Copy the generated token

3.2 Generate Docker Hub Access Token
Go to hub.docker.com → Account Settings → Security → New Access Token

Give it a name (e.g., "jenkins-pipeline")

Select permissions: "Read & Write"

Copy the token (starts with dckr_pat_)

3.3 Set Environment Variables
Add to 
 
    /etc/default/jenkins or /lib/systemd/system/jenkins.service:

bash

    export GITHUB_USERNAME="Praveena0308"
    export GITHUB_TOKEN="ghp_your_github_token_here"
    export DOCKER_USERNAME="pravee033"
    export DOCKER_TOKEN="dckr_pat_your_docker_token_here"

3.4 Verify Credentials in Jenkins
bash
# You can also verify via Jenkins CLI
java -jar jenkins-cli.jar -s http://localhost:8080/ list-credentials
Or check in UI: Manage Jenkins → Manage Credentials

4️⃣ SMEE.IO WEBHOOK CONFIGURATION

4.1 Why Smee.io?
When Jenkins runs on localhost or behind a firewall, GitHub cannot send webhooks directly. Smee.io acts as a tunnel:

text
GitHub → Smee.io (public) → Jenkins (localhost:8080)
4.2 Install Smee Client
bash
# Install Node.js and npm if not installed
sudo apt install nodejs npm -y

# Install smee-client globally
sudo npm install --global smee-client

# Verify installation
smee --version

4.3 Create a Smee Channel
Go to https://smee.io

Click "Start a new channel"

Copy your unique URL (e.g., https://smee.io/your-unique-channel-123)

4.4 Start Smee Client
bash
# Start smee client to forward webhooks to Jenkins
smee --url https://smee.io/your-unique-channel-123 \
     --path /github-webhook/ \
     --port 8080
Important: Keep this terminal window open! Or run as a service:

bash

     Create systemd service for smee
    sudo nano /etc/systemd/system/smee.service
ini
[Unit]
Description=Smee Webhook Forwarder
After=network.target jenkins.service

[Service]
Type=simple
User=jenkins
ExecStart=/usr/bin/smee --url 
        
    https://smee.io/your-unique-channel-123 --path /github-webhook/ --port 8080
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
bash
# Enable and start the service
sudo systemctl enable smee

sudo systemctl start smee

sudo systemctl status smee

4.5 Configure GitHub Webhook
Go to your GitHub repository: https://github.com/Praveena0308/voting-app-cicd

Click Settings → Webhooks → Add webhook

Configure:

Payload URL: https://smee.io/your-unique-channel-123

Content type: application/json

Events: "Just the push event"

Active: Checked

Click Add webhook

4.6 Test Webhook
bash
# Make a test push
git add .
git commit -m "Test webhook"
git push origin main

Check smee terminal - should show:
POST http://localhost:8080/github-webhook/ - 200


5️⃣ JENKINS PIPELINE DEVELOPMENT
5.1 Create Pipeline Job
Jenkins Dashboard → New Item

Enter name: voting-app-pipeline

Select Pipeline → OK

Configure:

Build Triggers: Check "GitHub hook trigger for GITScm polling"

Pipeline Definition: "Pipeline script from SCM"

SCM: Git

Repository URL: https://github.com/Praveena0308/voting-app-cicd.git

Credentials: github-credentials

Branches to build: */main

Script Path: Jenkinsfile

Click Save

5.2 Understanding Jenkinsfile Structure
groovy
pipeline {
    agent any  // Where to run
    
    environment {
        // Environment variables
    }
    
    triggers {
        // What triggers the pipeline
    }
    
    stages {
        stage('Checkout') {
            steps {
                // Get code
            }
        }
        
        stage('Build') {
            parallel {
                // Run in parallel
                stage('Service 1') {
                    agent {
                        docker {
                            // Run inside container
                        }
                    }
                    steps {
                        // Build steps
                    }
                }
            }
        }
    }
    
    post {
        success {
            // On success
        }
        failure {
            // On failure
        }
    }
}
7️⃣ TROUBLESHOOTING & COMMON ISSUES
Issue 1: Jenkins can't find Docker
bash
# Check if docker is installed
docker --version

# Check if jenkins user is in docker group
groups jenkins

# Fix:
sudo usermod -aG docker jenkins

sudo systemctl restart jenkins

Issue 2: "docker: not found" inside containers

Solution: Mount Docker socket and use Docker-in-Docker images


    groovy
    agent {
    docker {
        image 'docker:25.0-dind'
        reuseNode true
        args '-u root:root -v /var/run/docker.sock:/var/run/docker.sock'
     }
    }
Issue 3: Docker API version mismatch
text

Error response from daemon: client version 1.41 is too old.

Solution: Set explicit API version

bash
      
      export DOCKER_API_VERSION=1.44


Issue 4: Python externally-managed-environment error
text

error: externally-managed-environment

Solution: Use virtual environment

bash

     python3 -m venv /venv
    source /venv/bin/activate
    pip install -r requirements.txt

Issue 5: .NET runtime identifier issues
text

error NETSDK1047: Assets file doesn't have a target for 'net7.0/linux-x64'

Solution: Specify runtime in all commands

bash
dotnet restore -r linux-x64
dotnet build -r linux-x64
dotnet publish -r linux-x64
Issue 6: JCasC not applying
bash
# Check if file exists
ls -la /var/lib/jenkins/casc_configs/jenkins.yaml

# Check permissions
sudo ls -la /var/lib/jenkins/casc_configs/

# Check Jenkins logs
sudo journalctl -u jenkins | grep -i "casc \ | configuration"

# Test configuration
sudo -u jenkins java -jar /usr/share/jenkins/jenkins.war --version
Issue 7: Webhooks not triggering
bash
# Check smee is running
ps aux | grep smee

# Test webhook manually
curl -X POST -H "Content-Type: application/json" \
  -d '{"ref": "refs/heads/main"}' \
  http://localhost:8080/github-webhook/

# Check GitHub webhook deliveries
# GitHub repo → Settings → Webhooks → Recent Deliveries
8️⃣ KEY COMMANDS CHEAT SHEET
Jenkins Service Commands
bash
# Start/Stop/Restart
sudo systemctl start jenkins

sudo systemctl stop jenkins

sudo systemctl restart jenkins

sudo systemctl status jenkins


# View logs
sudo journalctl -u jenkins -f

sudo tail -f /var/log/jenkins/jenkins.log
JCasC Commands

bash
# Switch to jenkins user
sudo -u jenkins -s
cd $JENKINS_HOME

# Create config directory
sudo mkdir -p /var/lib/jenkins/casc_configs
sudo chown -R jenkins:jenkins /var/lib/jenkins/casc_configs

# Edit config
sudo nano /var/lib/jenkins/casc_configs/jenkins.yaml

# Set permissions
sudo chmod 600 /var/lib/jenkins/casc_configs/jenkins.yaml
Smee.io Commands
bash
# Install
npm install --global smee-client

# Run
smee --url https://smee.io/your-channel --path /github-webhook/ --port 8080

# Create service
sudo nano /etc/systemd/system/smee.service

sudo systemctl enable smee

sudo systemctl start smee

sudo systemctl status smee

Docker Commands

bash
# Add user to docker group
sudo usermod -aG docker jenkins

# Set API version
export DOCKER_API_VERSION=1.44

# Clean up
docker system prune -f
docker image prune -f
Git Commands
bash
# Clone repo
git clone https://github.com/Praveena0308/voting-app-cicd.git

# Add Jenkinsfile
git add Jenkinsfile
git commit -m "Add Jenkins pipeline"
git push origin main

# Check remote
git remote -v
git remote set-url origin https://github.com/Praveena0308/your-repo.git
Jenkins CLI Commands
bash
# Download CLI
wget http://localhost:8080/jnlpJars/jenkins-cli.jar

# List jobs
java -jar jenkins-cli.jar -s http://localhost:8080/ list-jobs

# Build job
java -jar jenkins-cli.jar -s http://localhost:8080/ build voting-app-pipeline

# Console output
java -jar jenkins-cli.jar -s http://localhost:8080/ console voting-app-pipeline 21



🎓 LEARNING SUMMARY
What You've Mastered:
Area	Skills
Jenkins	Installation, configuration, pipeline development
JCasC	Infrastructure as Code, credential management
Docker	Docker-in-Docker, image building, pushing to registry
Webhooks	Smee.io tunneling, GitHub integration
Multi-service	Parallel builds, Python/Node.js/.NET
Security	Credential management, secure tokens
Linux	Systemd services, user management, permissions
Your Pipeline Achievements:
✅ Automated builds on every git push

✅ Three microservices built in parallel

✅ Docker images created and versioned

✅ Images pushed to Docker Hub

✅ Full CI/CD pipeline with zero manual intervention

📝 FINAL NOTES
This documentation covers your entire journey from zero to a fully functional Jenkins CI/CD pipeline. Bookmark this for:

Revision - Before interviews

Reference - When building new pipelines

Troubleshooting - When issues arise

Teaching others - Share your knowledge

<div align="center"> <h2>🎉 Congratulations, DevOps Engineer! 🏆</h2> <p><strong>You've successfully built a complete CI/CD pipeline for microservices!</strong></p> <p>Document created: February 22, 2026</p> <p>Author: <a href="https://github.com/Praveena0308">Praveena0308</a></p> </div># Test Mon Feb 23 16:47:41 CET 2026
# Another test Mon Feb 23 16:49:48 CET 2026
# Final test Mon Feb 23 17:03:37 CET 2026
Final test Mon Feb 23 17:14:37 CET 2026
# Auto-trigger test Mon Feb 23 17:23:02 CET 2026
# Final automation test Mon Feb 23 17:31:08 CET 2026
