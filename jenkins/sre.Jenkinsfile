pipeline {
    agent {
        label 'runner'
    }

    stages {
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
