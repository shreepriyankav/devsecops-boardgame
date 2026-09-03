pipeline {
    agent any

    tools {
        jdk 'Java21'
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

                    withSonarQubeEnv('SonarQube') {
                        sh """
                            export PATH="${scannerHome}/bin:\\$PATH"
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

        stage('Publish To Nexus') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'nexus-credentials',
                        usernameVariable: 'NEXUS_USERNAME',
                        passwordVariable: 'NEXUS_PASSWORD'
                    )
                ]) {
                    sh '''
                        mvn deploy:deploy-file \
                          -DgroupId=com.javaproject \
                          -DartifactId=database_service_project \
                          -Dversion=0.0.5-SNAPSHOT \
                          -Dpackaging=jar \
                          -Dfile=target/database_service_project-0.0.5-SNAPSHOT.jar \
                          -DrepositoryId=nexus \
                          -Durl=http://172.31.27.218:8081/repository/boardgame-snapshots/
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t boardgame:0.0.5 .
                '''
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image boardgame:0.0.5'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        docker tag boardgame:0.0.5 $DOCKER_USERNAME/boardgame:0.0.5

                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin

                        docker push $DOCKER_USERNAME/boardgame:0.0.5

                        docker logout
                    '''
                }
            }
        }
    }
}

