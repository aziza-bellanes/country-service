
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
                            -Dsonar.junit.reportPaths=target/surefire-reports \
                            -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
                        """
                    }
                }
            }
        }

                
        stage('Deploy to Nexus') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'nexus-credentials',
                        usernameVariable: 'NEXUS_USER',
                        passwordVariable: 'NEXUS_PASS'
                    )
                ]) {
                    sh '''
                        set +x
                        set -eu
                        umask 077
        
                        SETTINGS_FILE=$(mktemp)
                        trap 'rm -f "$SETTINGS_FILE"' EXIT
                        export SETTINGS_FILE
        
                        python3 -c 'import os, xml.etree.ElementTree as ET; root = ET.Element("settings", xmlns="http://maven.apache.org/SETTINGS/1.2.0"); server = ET.SubElement(ET.SubElement(root, "servers"), "server"); ET.SubElement(server, "id").text = "nexus-snapshots"; ET.SubElement(server, "username").text = os.environ["NEXUS_USER"]; ET.SubElement(server, "password").text = os.environ["NEXUS_PASS"]; ET.ElementTree(root).write(os.environ["SETTINGS_FILE"], encoding="utf-8", xml_declaration=True)'
        
                        mvn -s "$SETTINGS_FILE" deploy -DskipTests
                    '''
                }
            }
        }



    }
}
