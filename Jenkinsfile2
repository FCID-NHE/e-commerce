pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/FCID-NHE/e-commerce.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("FCID-NHE/e-commerce:${env.BUILD_ID}")
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials') {
                        dockerImage.push("${env.BUILD_ID}")
                        dockerImage.push("latest")
                    }
                }
            }
        }
        stage('Deploy to Remote Docker Server') {
            steps {
                script {
                    sh 'ssh yoda@10.10.10.10 "docker pull FCID-NHE/e-commerce:${env.BUILD_ID}"'
                    sh 'ssh yoda@10.10.10.10 "docker stop my-container || true"'
                    sh 'ssh yoda@10.10.10.10 "docker rm my-container || true"'
                    sh 'ssh yoda@10.10.10.10 "docker run -d --name my-container -p 8080:80 FCID-NHE/e-commerce:${env.BUILD_ID}"'
                }
            }
        }
        stage('Update GitHub Status') {
            steps {
                script {
                    def status = currentBuild.result == 'SUCCESS' ? 'success' : 'failure'
                    step([$class: 'GitHubCommitStatusSetter', contextSource: [$class: 'ManuallyEnteredCommitContextSource', context: 'ci/build'], 
                          statusResultSource: [$class: 'ConditionalStatusResultSource', results: [[status: status, state: status]]]])
                }
            }
        }
    }
    post {
        always {
            emailext (
                to: 'user@example.com',
                subject: "Build ${currentBuild.fullDisplayName}",
                body: "Build ${currentBuild.fullDisplayName} finished with status: ${currentBuild.result}"
            )
        }
    }
}
