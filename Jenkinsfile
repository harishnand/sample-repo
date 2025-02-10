pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                script {
                    echo "Building project for branch: ${env.BRANCH_NAME}"
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def deployPath = "/home/ubuntu/deployments/${env.BRANCH_NAME}"
                    sh """
                    ssh -o StrictHostKeyChecking=no ubuntu@176.34.98.123 'mkdir -p ${deployPath}'
                    scp -o StrictHostKeyChecking=no app.py ubuntu@176.34.98.123:${deployPath}/
                    ssh -o StrictHostKeyChecking=no ubuntu@176.34.98.123 'python3 ${deployPath}/app.py &'
                    """
                }
            }
        }

        stage('Notify Telegram') {
            steps {
                script {
                    def commitAuthor = sh(script: "git log -1 --pretty=format:'%an'", returnStdout: true).trim()
                    def commitId = sh(script: "git log -1 --pretty=format:'%h'", returnStdout: true).trim()
                    def commitMessage = sh(script: "git log -1 --pretty=format:'%s'", returnStdout: true).trim()
                    def jobStatus = currentBuild.result ?: 'SUCCESS'
                    def telegramMessage = """
                    Hi, Jenkins job: ${env.JOB_NAME} status is ${jobStatus}
                    env: origin/${env.BRANCH_NAME}
                    Committed by: ${commitAuthor}
                    commit-id: ${commitId}
                    commit msg: ${commitMessage}
                    """

                    withCredentials([string(credentialsId: 'telegram-token', variable: 'TELEGRAM_TOKEN')]) {
                        sh """
                        curl -s -X POST https://api.telegram.org/bot${TELEGRAM_TOKEN}/sendMessage -d chat_id=-4689567738 -d text="${telegramMessage}"
                        """
                    }
                }
            }
        }
    }
}
