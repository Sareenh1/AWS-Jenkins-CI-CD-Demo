# CI/CD Pipeline for Dockerized Application Deployment

This repository demonstrates a CI/CD pipeline implemented using Jenkins to automate the process of building, testing, pushing a Docker image to Amazon ECR, and deploying it to an EC2 instance. Below is a detailed explanation of the pipeline script and its stages.

## Prerequisites
1. **Jenkins Server**: A Jenkins instance must be set up and configured.
2. **AWS ECR**: Amazon Elastic Container Registry (ECR) must be created.
3. **AWS EC2 Instance**: An EC2 instance should be running and accessible via SSH.
4. **AWS CLI**: AWS CLI must be configured with the appropriate IAM permissions.
5. **Docker**: Docker should be installed and running on the Jenkins server and the EC2 instance.
6. **GitHub Repository**: The application code must be hosted in a GitHub repository.
7. **Jenkins Credentials**:
    - `github-creds`: For authenticating with the GitHub repository.
    - AWS credentials for accessing ECR.

## Pipeline Stages

### 1. **Clone Repo**
This stage pulls the latest code from the specified GitHub repository.
```groovy
stage('Clone Repo') {
    steps {
        git credentialsId: 'github-creds', url: 'https://github.com/Sareenh1/AWS-Jenkins-CI-CD-Demo.git'
    }
}
```
- **Details**:
    - `credentialsId`: Specifies the Jenkins credential ID for GitHub authentication.
    - `url`: URL of the GitHub repository.

### 2. **Build Docker Image**
This stage builds a Docker image for the application using the `Dockerfile` in the repository.
```groovy
stage('Build Docker Image') {
    steps {
        sh 'docker build -t jenkins-cicd-demo:v1 .'
    }
}
```
- **Details**:
    - The `docker build` command tags the image as `jenkins-cicd-demo:v1`.

### 3. **Run Unit Tests**
This stage runs unit tests for the application.
```groovy
stage('Run Unit Tests') {
    steps {
        sh 'npm test'
    }
}
```
- **Details**:
    - Assumes `npm` and the test suite are configured in the application.

### 4. **Push to ECR**
This stage pushes the Docker image to Amazon ECR.
```groovy
stage('Push to ECR') {
    steps {
        script {
            sh '''
            aws ecr get-login-password --region ap-south-1 | sudo docker login --username AWS --password-stdin 992382465189.dkr.ecr.ap-south-1.amazonaws.com
            sudo docker tag jenkins-cicd-demo:v1 992382465189.dkr.ecr.ap-south-1.amazonaws.com/jenkins-cicd-demo:v1
            sudo docker push 992382465189.dkr.ecr.ap-south-1.amazonaws.com/jenkins-cicd-demo:v1
            '''
        }
    }
}
```
- **Details**:
    - Logs into the ECR registry.
    - Tags the image with the ECR repository URI.
    - Pushes the image to the specified ECR repository.

### 5. **Deploy to EC2**
This stage deploys the Docker container on an EC2 instance.
```groovy
stage('Deploy to EC2') {
    steps {
        sh '''
        ssh -o StrictHostKeyChecking=no ubuntu@3.110.167.111 "
        docker pull 992382465189.dkr.ecr.ap-south-1.amazonaws.com/jenkins-cicd-demo:v1 &&
        docker run -d -p 3000:3000 992382465189.dkr.ecr.ap-south-1.amazonaws.com/jenkins-cicd-demo:v1
        "
        '''
    }
}
```
- **Details**:
    - Uses SSH to connect to the EC2 instance.
    - Pulls the Docker image from ECR.
    - Runs the container on port `3000`.

### 6. **Validation**
This stage validates the deployment by checking the application's availability.
```groovy
stage('Validation') {
    steps {
        sh 'curl -I http://3.110.167.111:3000'
    }
}
```
- **Details**:
    - Sends an HTTP request to the application URL to confirm it is running.

### 7. **Post Actions**
This section sends a notification email after the pipeline execution.
```groovy
post {
    always {
        emailext(
            subject: "Build Status: ${currentBuild.currentResult}",
            body: "Build #${BUILD_NUMBER} Status: ${currentBuild.currentResult}",
            to: 'Sareenh10@gmail.com'
        )
    }
}
```
- **Details**:
    - Sends an email with the build status and build number.

## Configuration Instructions

### AWS CLI Configuration
1. Install the AWS CLI on the Jenkins server.
2. Configure the CLI with the required credentials and region:
   ```bash
   aws configure
   ```

### Jenkins Configuration
1. Install necessary plugins:
    - Git Plugin
    - Docker Pipeline Plugin
    - Email Extension Plugin
2. Add credentials:
    - Add GitHub credentials (`github-creds`).
    - Add AWS credentials if required.
3. Create a pipeline job and paste the above script.

### EC2 Instance Configuration
1. Install Docker on the EC2 instance:
   ```bash
   sudo apt update && sudo apt install -y docker.io
   ```
2. Ensure the EC2 instance's security group allows inbound traffic on port `3000`.

## Testing the Pipeline
1. Trigger the pipeline manually or configure a webhook for automatic triggers.
2. Monitor the stages for any errors.
3. Verify the deployment by accessing `http://3.110.167.111:3000` in a browser.

## Conclusion
This pipeline automates the entire process from code pull to deployment, ensuring faster and more reliable delivery of updates. The steps provided can be customized further to meet specific requirements.

