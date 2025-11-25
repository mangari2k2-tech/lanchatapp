pipeline {
    agent any
    
    tools {
        jdk 'JDK11'
    }
    
    environment {
        SONAR_TOKEN = credentials('sonar-token')
        SONAR_HOST_URL = 'http://sonarqube:9000'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                script {
                    sh 'ant clean compile jar'
                }
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                script {
                    sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=lanchatapp \
                        -Dsonar.sources=src \
                        -Dsonar.host.url=${SONAR_HOST_URL} \
                        -Dsonar.login=${SONAR_TOKEN} \
                        -Dsonar.java.binaries=build/classes
                    '''
                }
            }
        }
        
        stage('Trivy Security Scan') {
            steps {
                script {
                    sh '''
                        docker run --rm -v $(pwd):/workspace aquasec/trivy:latest fs /workspace --format json --output trivy-report.json
                        docker run --rm -v $(pwd):/workspace aquasec/trivy:latest fs /workspace --format table
                    '''
                }
            }
        }
        
        stage('Archive Results') {
            steps {
                archiveArtifacts artifacts: 'dist/*.jar, trivy-report.json', fingerprint: true
                publishHTML([
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: '.',
                    reportFiles: 'trivy-report.json',
                    reportName: 'Trivy Security Report'
                ])
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
