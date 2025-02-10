pipeline {
    agent any

    environment {
        EC2_HOST = "176.34.98.123"  // Your EC2 instance
        DEPLOY_DIR = "/home/ubuntu/deployments/${env.BRANCH_NAME}"  // Deployment path
        TELEGRAM_BOT_TOKEN = credentials('telegram-token')  // Ensure Telegram token is in Jenkins credentials
        TELEGRAM_CHAT_ID = "-4689567738"  // Your Telegram chat ID (update this)
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    echo "Checking out code from branch: ${env.BRANCH_NAME}"
                    checkout scm
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    echo "Building project for branch: ${env.BRANCH_NAME}"
                    sh "echo 'Build successful!'"
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-key', keyFileVariable: 'SSH_KEY')]) {
                        sh """
                        echo "Deploying to EC2 instance..."
                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no ubuntu@$EC2_HOST 'mkdir -p $DEPLOY_DIR'
                        scp -i $SSH_KEY -o StrictHostKeyChecking=no app.py ubuntu@$EC2_HOST:$DEPLOY_DIR/
                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no ubuntu@$EC2_HOST 'nohup python3 $DEPLOY_DIR/app.py > $DEPLOY_DIR/app.log 2>&1 &'
                        echo "Deployment completed."
                        """
                    }
                }
            }
        }

        stage('Notify Telegram') {
            steps {
                script {
                    def git_commit = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    def commit_msg = sh(script: "git log -1 --pretty=%B", returnStdout: true).trim()
                    def committer = sh(script: "git log -1 --pretty=format:'%an'", returnStdout: true).trim()

                    def message = """
                    Hi, Jenkins job: *${JOB_NAME}* status is *${currentBuild.currentResult}*
                    Env: *${env.GIT_BRANCH}*
                    Committed by: *${committer}*
                    Commit ID: *${git_commit}*
                    Commit Msg: *${commit_msg}*
                    """

                    sh """
                    curl -s -X POST https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/sendMessage \\
                    -d chat_id=$TELEGRAM_CHAT_ID -d text="$message" -d parse_mode=Markdown
                    """
                }
            }
        }
    }

    post {
        failure {
            script {
                sh """
                curl -s -X POST https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/sendMessage \\
                -d chat_id=$TELEGRAM_CHAT_ID -d text="Jenkins job *${JOB_NAME}* failed on branch *${env.GIT_BRANCH}*" -d parse_mode=Markdown
                """
            }
        }
    }
}
