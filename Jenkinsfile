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
                    def isMerged = sh(script: '''
                        git log --merges -n 1 --pretty=format:'%s' | grep 'Merge branch' || true
                    ''', returnStdout: true).trim()

                    if (isMerged.contains('Merge branch')) {
                        echo "Merge detected! Running terraform apply..."
                        sh '''
                        terraform apply -auto-approve tfplan
                        '''
                        slackNotification("Terraform apply successful! ")
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
                slackNotification("Terraform deployment successful! ")
            }
        }
        failure {
            script {
                slackNotification("Terraform deployment failed! ")
            }
        }
    }
}

def slackNotification(message) {
    slackSend(channel: '#jenkins', message: message)
}
