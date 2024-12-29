pipeline {

    agent any

    environment {
        registry = "sathish0205/vprofileapp"
        registryCredentials = "dockerhub"
    }

    stages {

        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage('Unit Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Integration Test') {
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }

        stage('Code Analysis with Checkstyle') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        stage('Code Analysis with SonarQube') {
    environment {
        scannerHome = tool 'mysonarscanner4'
        SONAR_SCANNER_OPTS = "--add-opens java.base/java.lang=ALL-UNNAMED --add-opens java.base/java.lang.reflect=ALL-UNNAMED"
    }
    steps {
        withSonarQubeEnv('sonar-pro') {
            sh '''${scannerHome}/bin/sonar-scanner \
               -Dsonar.projectKey=vprofile \
               -Dsonar.projectName=vprofile-repo \
               -Dsonar.projectVersion=1.0 \
               -Dsonar.sources=src/ \
               -Dsonar.java.binaries=target/classes \
               -Dsonar.junit.reportsPath=target/surefire-reports/ \
               -Dsonar.jacoco.reportsPath=target/jacoco.exec \
               -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
        }
        timeout(time: 10, unit: 'MINUTES') {
            waitForQualityGate abortPipeline: true
        }
    }
}

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build "${registry}:V${BUILD_NUMBER}"
                }
            }
        }

        stage('Upload Docker Image') {
            steps {
                script {
                    docker.withRegistry('', registryCredentials) {
                        dockerImage.push("V${BUILD_NUMBER}")
                        dockerImage.push("latest")
                    }
                }
            }
        }

        stage('Remove Docker Images') {
            steps {
                sh "docker rmi ${registry}:V${BUILD_NUMBER}"
            }
        }

        stage('Kubernetes Deployment') {
            agent { label 'KOPS' }
            steps {
                sh "helm upgrade --install --force vprofile-stack helm/vprofilecharts --set appimage=${registry}:V${BUILD_NUMBER} --namespace prod"
            }
        }

    }
}