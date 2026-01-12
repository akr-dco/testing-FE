pipeline {
    agent any

    environment {
        TARGET_HOST = "192.168.192.88"
    }

    stages {

        stage('Resolve Target Environment') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        env.TARGET_USER = "apps"
                        env.TARGET_DIR  = "/home/apps/cicd/apps-fe"
                        env.ENV_NAME    = "PRODUCTION"
                    } 
                    else if (env.BRANCH_NAME == 'staging') {
                        env.TARGET_USER = "dev"
                        env.TARGET_DIR  = "/home/dev/cicd-dev/apps-fe"
                        env.ENV_NAME    = "STAGING"
                    } 
                    else {
                        error "❌ Branch ${env.BRANCH_NAME} is not allowed to deploy"
                    }
                }

                echo "Environment : ${env.ENV_NAME}"
                echo "Branch      : ${env.BRANCH_NAME}"
                echo "Target      : ${env.TARGET_USER}@${TARGET_HOST}:${env.TARGET_DIR}"
            }
        }

        stage('Prepare Target') {
            steps {
                sshagent(['privatekey']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${TARGET_USER}@${TARGET_HOST} '
                        mkdir -p ${TARGET_DIR}
                    '
                    """
                }
            }
        }

        stage('Sync Repository') {
            steps {
                sshagent(['privatekey']) {
                    sh """
                    rsync -avz --delete \
                        --exclude '.git' \
                        --exclude '.jenkins' \
                        ./ \
                        ${TARGET_USER}@${TARGET_HOST}:${TARGET_DIR}/
                    """
                }
            }
        }

        stage('Deploy Docker Compose') {
            steps {
                sshagent(['privatekey']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${TARGET_USER}@${TARGET_HOST} '
                        cd ${TARGET_DIR} &&
                        docker compose up -d --build --force-recreate --remove-orphans
                    '
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ ${env.ENV_NAME} Deployment SUCCESS"
        }
        failure {
            echo "❌ ${env.ENV_NAME} Deployment FAILED"
        }
    }
}
