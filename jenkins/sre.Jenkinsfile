pipeline {
    agent {
        label 'runner'
    }
    parameters {
        string(name: 'NODE_EXPORTER_HOST', defaultValue: 'node-exporter-host', description: 'The IP or hostname for Node Exporter')
        string(name: 'NODE_EXPORTER_USER', defaultValue: 'user', description: 'SSH username for Node Exporter host')
        string(name: 'NODE_EXPORTER_PASSWORD', defaultValue: 'password', description: 'SSH password for Node Exporter host')
    }

    stages {
        stage('Install Prometheus, Grafana, and Node Exporter') {
            steps {
                script {
                    sh 'docker compose up -d --build'
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
