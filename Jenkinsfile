pipeline {
    agent any

    tools {
        jdk 'Java11'
        maven 'Maven3.9.12'
    }

    stages {
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs .'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }
    }
}
