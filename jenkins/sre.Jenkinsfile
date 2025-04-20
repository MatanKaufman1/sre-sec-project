pipeline {
    agent {
        label 'runner'
    }

    environment {
        SLACK_CHANNEL = '#all-sre'
    }

    stages {
        stage('Install yamlLinter') {
            steps {
                script {
                    sh '''
                        sudo apt update
                        sudo apt install yamllint -y
                    '''
                }
            }
        }

        stage('Check yaml files') {
            steps {
                script {
                    sh 'yamllint .'
                }
            }
        }

        stage('Install checkov') {
            steps {
                script {
                    sh '''
                        sudo apt update
                        sudo apt install python3-pip -y
                        sudo pip install --upgrade pip
                        sudo pip install checkov
                    '''
                }
            }
        }

        stage('Run checkov') {
            steps {
                script {
                    sh 'checkov -d . -o json'
                }
            }
        }

        stage('Install Prometheus, Grafana, and Node Exporter') {
            steps {
                script {
                    sh '''
                        cd prometheus-grafana/
                        docker compose up -d --build
                    '''
                }
            }
        }

        stage('Check Running Containers') {
            steps {
                script {
                    sh 'docker ps'
                }
            }
        }
    }

    post {
        success {
            withCredentials([string(credentialsId: 'slack', variable: 'SLACK_WEBHOOK_URL')]) {
                sh """
                    curl -X POST --data-urlencode 'payload={
                        "channel": "${SLACK_CHANNEL}",
                        "text": "✅ Build succeeded! All stages passed.",
                    }' $SLACK_WEBHOOK_URL
                """
            }
        }

        failure {
            withCredentials([string(credentialsId: 'slack', variable: 'SLACK_WEBHOOK_URL')]) {
                sh """
                    curl -X POST --data-urlencode 'payload={
                        "channel": "${SLACK_CHANNEL}",
                        "text": "❌ Build failed! Check the Jenkins logs.",
                    }' $SLACK_WEBHOOK_URL
                """
            }
        }
    }
}
