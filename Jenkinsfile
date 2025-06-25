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
        ACR_LOGIN_SERVER = 'dockerregnodejs.azurecr.io'
        ACR_CREDENTIALS_ID = 'acr-credentials'
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
                        script {
                            mvnTest()
                        }
                    }
                }
                stage('Maven Compile') {
                    steps {
                        script {
                            mvnCompile()
                        }
                    }
                }
                stage('Maven Package') {
                    steps {
                        script {
                            mvnPackage()
                        }
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
                    dockerBuildAndPush(
                        'petclinic', 
                        "${env.BUILD_NUMBER}", 
                        env.ACR_LOGIN_SERVER, 
                        '', 
                        env.ACR_CREDENTIALS_ID
                    )
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
            script {
                log("Petclinic Pipeline completed successfully")
            }
        }
        failure {
            script {
                log("Petclinic Pipeline failed — check logs")
            }
        }
    }
}
