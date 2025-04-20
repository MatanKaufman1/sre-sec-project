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
        slackSend (
            channel: '#succeeded-build',
            message: "✅ pipeline succeeded.",
            tokenCredentialId: 'slack',   // Make sure this is the same credential ID as used in Freestyle
            teamDomain: 'sre-wcs3027'      // Optional if you configured it globally in Jenkins
        )
    }
    failure {
        slackSend (
            channel: '#devops-alerts',
            message: "❌ pipeline failed.",
            tokenCredentialId: 'slack',
            teamDomain: 'sre-wcs3027'
        )
    }
}

