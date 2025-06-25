@Library('my-shared-library') _

pipeline {
    agent any
    tools {
        maven 'maven'
    }
    environment {
        IMAGE_NAME = 'myrepo/myapp'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        SCANNER_HOME = tool 'Sonar-scanner'
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
        sonarScan(params: '''
            -Dsonar.organization=bkrrajmali
            -Dsonar.projectName=petclinic
            -Dsonar.projectKey=bkrrajmali_petclinic
            -Dsonar.java.binaries=target/classes
            -Dsonar.exclusions=**/trivy-report.txt
            ''')
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

        // stage('Docker Build & Push') {
        //     steps {
        //         dockerBuildPush(env.IMAGE_NAME, env.IMAGE_TAG)
        //     }
        // }

        // stage('Trivy Scan') {
        //     steps {
        //         trivyScan("${env.IMAGE_NAME}:${env.IMAGE_TAG}")
        //     }
        // }
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

