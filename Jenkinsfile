@Library('my-shared-library') _

pipeline {
    agent any

    environment {
        IMAGE_NAME = 'myrepo/myapp'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                log("Code checkout completed.")
            }
        }

        stage('Maven Test') {
            steps {
                mvnStage("test")
            }
        }

        stage('Maven Compile') {
            steps {
                mvnStage("compile")
            }
        }

        stage('Maven Package') {
            steps {
                mvnStage("package")
            }
        }

        stage('SonarQube Scan') {
            steps {
                sonarScan()
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
                dockerBuildPush(env.IMAGE_NAME, env.IMAGE_TAG)
            }
        }

        stage('Trivy Scan') {
            steps {
                trivyScan("${env.IMAGE_NAME}:${env.IMAGE_TAG}")
            }
        }
    }

    post {
        success {
            log("Pipeline completed successfully.")
        }
        failure {
            log("Pipeline failed.")
        }
    }
}

