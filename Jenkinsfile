
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

        
        stage('Publish Reports to Nexus') {
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
        
                        REPO_URL="http://localhost:8081/repository/ci-reports"
                        DEST="country-service/build-${BUILD_NUMBER}"
        
                        test -f target/site/jacoco/index.html
                        test -f target/site/jacoco/jacoco.xml
                        test -d target/surefire-reports

                        find target/surefire-reports -name 'TEST-*.xml' -type f | grep -q .
        
                        find target/site/jacoco -type f | while IFS= read -r file; do
                            relative_path=${file#target/site/jacoco/}
                            curl --fail --silent --show-error \
                                --user "$NEXUS_USER:$NEXUS_PASS" \
                                --upload-file "$file" \
                                "$REPO_URL/$DEST/jacoco/$relative_path"
                        done
        
                        find target/surefire-reports -name '*.xml' -type f | while IFS= read -r file; do
                            filename=${file##*/}
                            curl --fail --silent --show-error \
                                --user "$NEXUS_USER:$NEXUS_PASS" \
                                --upload-file "$file" \
                                "$REPO_URL/$DEST/surefire/$filename"
                        done
                    '''
                }
            }
        }




    }
}
