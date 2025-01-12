pipeline {
    agent any
    stages {
        stage('Clone Repo') {
            steps {
                git credentialsId: 'github-creds', url: 'https://github.com/Sareenh1/AWS-Jenkins-CI-CD-Demo.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t jenkins-cicd-demo:v1 .'
            }
        }
        stage('Run Unit Tests') {
            steps {
                sh 'npm test'
            }
        }
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
        stage('Deploy to EC2') {
            steps {
                sh '''
                ssh -o StrictHostKeyChecking=no ec2-user@3.110.167.111 "
                docker pull 992382465189.dkr.ecr.ap-south-1.amazonaws.com/jenkins-cicd-demo:v1 &&
                docker run -d -p 3000:3000 992382465189.dkr.ecr.ap-south-1.amazonaws.com/jenkins-cicd-demo:v1
                "
                '''
            }
        }
        stage('Validation') {
            steps {
                sh 'curl -I http://3.110.167.111:3000'
            }
        }
    }
    post {
        always {
            emailext(
                subject: "Build Status: ${currentBuild.currentResult}",
                body: "Build #${BUILD_NUMBER} Status: ${currentBuild.currentResult}",
                to: 'team@example.com'
            )
        }
    }
}

