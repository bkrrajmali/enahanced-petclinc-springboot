@Library('my-shared-library') _

pipeline {
    agent any

    tools {
        maven 'maven' // Defined in Jenkins Global Tool Config
    }

    environment {
        IMAGE_NAME = 'bkrrajmali/petclinic'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        SCANNER_HOME = tool 'Sonar-scanner' // Optional if you're not using CLI scanner
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                log("Code checked out")
            }
        }

    stage('Parallel Maven Build') {
            parallel {
                stage('Maven Test') {
                    steps {
                        mvnStage('test')
                    }
                }
                stage('Maven Compile') {
                    steps {
                        mvnStage('compile')
                    }
                }
                stage('Maven Package') {
                    steps {
                        mvnStage('package')
                    }
                }
            }
        }

        stage('SonarQube Scan') {
            steps {
                sonarScan(params: '''
                    -Dsonar.login=$SONAR_TOKEN
                    -Dsonar.organization=bkrrajmali
                    -Dsonar.projectKey=bkrrajmali_petclinic
                    -Dsonar.projectName=petclinic
                    -Dsonar.java.binaries=target/classes
                    -Dsonar.exclusions=**/trivy-report.txt
                ''')
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
            log("✅ Petclinic Pipeline executed successfully")
        }
        failure {
            log("❌ Petclinic Pipeline failed — check logs")
        }
    }
}
