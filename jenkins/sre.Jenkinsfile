pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building..'
                ecgo "hello world"
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}
