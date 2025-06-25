@Library('my-shared-library') _

pipeline {
    agent any

    tools {
        maven 'maven'
    }

    environment {
        IMAGE_NAME = 'bkrrajmali/petclinic'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        SCANNER_HOME = tool 'Sonar-scanner'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build - Maven Tasks in Parallel') {
            parallel {
                stage('Maven Test') {
                    steps {
                        sh 'mvn test'
                    }
                }
                stage('Maven Compile') {
                    steps {
                        sh 'mvn compile'
                    }
                }
                stage('Maven Package') {
                    steps {
                        sh 'mvn package -DskipTests'
                    }
                }
            }
        }

        stage('SonarQube Scan') {
            steps {
                script {
                    sonarScan([
                        projectKey      : 'bkrrajmali_petclinic',
                        organization    : 'bkrrajmali',
                        projectName     : 'petclinic',
                        exclusions      : '**/trivy-report.txt'
                    ])
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    dockerBuildAndPush("${IMAGE_NAME}", "${IMAGE_TAG}")
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                script {
                    trivyScan("${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }
    }

    post {
        success {
            echo "[LOG] ✅ Petclinic Pipeline completed successfully"
        }
        failure {
            echo "[LOG] ❌ Petclinic Pipeline failed — check logs"
        }
    }
}
