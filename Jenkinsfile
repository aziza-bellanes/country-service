
pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    stages {
        stage('Checkout code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/aziza-bellanes/country-service.git'
            }
        }

        stage('Compile code') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Test code') {
            steps {
                sh 'mvn test'
            }

            post {
                success {
                    junit allowEmptyResults: true,
                          testResults: '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package code') {
            steps {
                sh 'mvn verify'
            }
        }

        
        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarQubeScanner'
        
                    withSonarQubeEnv('SonarQube') {
                        sh """
                            "${scannerHome}/bin/sonar-scanner" \
                            -Dsonar.projectKey=country-service \
                            -Dsonar.projectName=country-service \
                            -Dsonar.sources=src/main/java \
                            -Dsonar.java.binaries=target/classes \
                            -Dsonar.tests=src/test/java \
                            -Dsonar.junit.reportPaths=target/surefire-reports
                        """
                    }
                }
            }
        }

    }
}
