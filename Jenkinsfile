pipeline {
    agent any
    environment {
        TELEGRAM_BOT_TOKEN = credentials('TelegramToken')
    }
    stages {
        stage('Build') {
            steps {
                echo "Building..."
            }
        }
        stage('Deploy') {
            steps {
                echo "Deploying..."
            }
        }
    }
    post {
        success {
            script {
                def branchName = env.GIT_BRANCH
                def commitId = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                def commitMsg = sh(script: "git log -1 --pretty=%B", returnStdout: true).trim()
                def committer = sh(script: "git log -1 --pretty=format:'%an'", returnStdout: true).trim()

                def message = """
                Hi, Jenkins job: ${env.JOB_NAME} status is SUCCESS
                env: ${branchName}
                Committed by: ${committer}
                commit-id: ${commitId}
                commit msg: ${commitMsg}
                """

                sh """
                curl -s -X POST https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage -d chat_id=-4689567738 -d text="${message}"
                """
            }
        }
        failure {
            script {
                def message = "Jenkins job ${env.JOB_NAME} FAILED!"
                sh """
                curl -s -X POST https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage -d chat_id=-4689567738 -d text="${message}"
                """
            }
        }
    }
}
