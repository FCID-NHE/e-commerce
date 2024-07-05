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
