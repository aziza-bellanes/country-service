
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
                bat 'mvn clean compile'
            }
        }

        stage('Test code') {
            steps {
                bat 'mvn test'
            }
        }

        stage('Package code') {
            steps {
                bat 'mvn package -DskipTests'
            }
        }
    }

    post {
        always {
            junit 'target/surefire-reports/*.xml'
        }
    }
}
