pipeline {
    agent {
        label 'runner'
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
                    sh '''
                        yamllint .
                    '''
                }
            }
        }
        stage('install checkov') {
            steps {
               script {
                sh '''
                    apt update
                    apt install python3-pip -y
                    pip install --upgrade pip
                    pip install checkov
                    '''
               }
            }
        }
        stage ('run checkov') {
            steps {
                script {
                    sh '''
                          echo this run checkov
                          '''
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
}
