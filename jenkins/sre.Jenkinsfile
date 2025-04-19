pipeline {
    agent {
        label 'runner'
    }

    stages {
        stage ( 'Install yamlLinter')
            steps {
                script {
                    sh '''
                        sudo apt update
                        sudo apt install yamllint
                        '''
                }
            }
         stage ('check yaml files')
            steps {
                scripts {
                    sh '''
                        yamllint -r .
                        '''
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
