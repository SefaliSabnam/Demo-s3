pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID_1')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_ACCESS_KEY_ID_1')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Terraform Init and Plan') {
            steps {
                script {
                    sh '''
                    terraform init
                    terraform plan -out=tfplan
                    '''
                }
            }
        }

        stage('Merge Check and Apply') {
            when {
                branch 'main'
            }
            steps {
                script {
                    def mergeCheck = sh(script: '''
                        git rev-list -n 1 --ancestry-path HEAD^2..HEAD || true
                    ''', returnStdout: true).trim()

                    if (mergeCheck) {
                        echo "Merge detected! Running terraform apply..."
                        sh '''
                        terraform apply -auto-approve tfplan
                        '''
                        slackNotification("Terraform apply successful! ✅")
                    } else {
                        echo "No merge detected. Skipping terraform apply."
                    }
                }
            }
        }
    }

    post {
        success {
            script {
                slackNotification("Terraform deployment successful! ✅")
            }
        }
        failure {
            script {
                slackNotification("Terraform deployment failed! ❌")
            }
        }
    }
}

def slackNotification(message) {
    slackSend(channel: '#jenkins', message: message)
}
