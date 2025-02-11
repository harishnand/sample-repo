pipeline {
    agent any
    environment {
        TELEGRAM_BOT_TOKEN = credentials('telegram-token')
        SSH_KEY = credentials('ec2-ssh-key')
        DEPLOY_USER = "ubuntu"
        DEPLOY_HOST = "176.34.98.123"
        DEPLOY_PATH = "/home/ubuntu/deployments"
    }
    
    stages {
        stage('Checkout') {
            steps {
                script {
                    BRANCH_NAME = env.BRANCH_NAME
                    echo "Checking out code from branch: ${BRANCH_NAME}"
                }
                checkout scm
            }
        }

        stage('Build') {
            steps {
                script {
                    echo "Building project for branch: ${BRANCH_NAME}"
                    sh "echo Build successful!"
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo "Deploying to EC2 instance..."
                    
                    // Ensure the SSH key is available
                    withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-key', keyFileVariable: 'SSH_KEY_PATH')]) {
                        sh """
                            chmod 400 ${SSH_KEY_PATH}
                            ssh -i ${SSH_KEY_PATH} -o StrictHostKeyChecking=no ${DEPLOY_USER}@${DEPLOY_HOST} "mkdir -p ${DEPLOY_PATH}/${BRANCH_NAME}"
                            scp -i ${SSH_KEY_PATH} -o StrictHostKeyChecking=no -r * ${DEPLOY_USER}@${DEPLOY_HOST}:${DEPLOY_PATH}/${BRANCH_NAME}
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            script {
                def commitDetails = sh(script: "git log -1 --pretty=format:'%H%n%an%n%s'", returnStdout: true).trim().split('\n')
                def commitId = commitDetails[0].take(8)
                def committer = commitDetails[1]
                def commitMessage = commitDetails[2]
                
                sh """
                    curl -s -X POST "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage" \\
                    -d chat_id="-4689567738" \\
                    -d text="✅ Hi, Jenkins job: *${env.JOB_NAME}* status is *SUCCESS*%0A🌍 env: *origin/${BRANCH_NAME}*%0A👤 Committed by: *${committer}*%0A🔗 commit-id: *${commitId}*%0A📝 commit msg: *${commitMessage}*" \\
                    -d parse_mode="Markdown"
                """
            }
        }

        failure {
            sh """
                curl -s -X POST "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage" \\
                -d chat_id="-4689567738" \\
                -d text="❌ Jenkins job *${env.JOB_NAME}* failed on branch *${BRANCH_NAME}*" \\
                -d parse_mode="Markdown"
            """
        }
    }
}
