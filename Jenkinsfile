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

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarQube Scanner'
            def java21Home = tool 'Java21'

            withSonarQubeEnv('SonarQube') {
                sh """
                    export JAVA_HOME="${java21Home}"
                    export PATH="${java21Home}/bin:${scannerHome}/bin:\$PATH"

                    java -version
                    sonar-scanner
                """
            }
        }
    }
}
        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }
    }
}
