pipeline {
    agent any

    tools {
        maven 'Maven-3'
    }

    environment {
        DOCKER_IMAGE = 'java-hello-world'
    }

    stages {
        stage('Git Checkout') {
            steps {
                echo 'Code already checked out by Jenkins SCM'
            }
        }

        stage('Unit Test & Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Sonar Scan') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.9.1.2184:sonar -Dsonar.java.binaries=target/classes'
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:$BUILD_NUMBER .'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f k8s/deployment.yaml'
                sh 'kubectl apply -f k8s/service.yaml'
            }
        }
    }

    post {
        always {
            junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
            publishHTML(target: [
                allowMissing: true,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: 'target/site/jacoco',
                reportFiles: 'index.html',
                reportName: 'Code Coverage Report'
            ])
        }
        success {
            mail to: 'suzannedsouza100@gmail.com',
                 subject: "SUCCESS: Build #${env.BUILD_NUMBER} - ${env.JOB_NAME}",
                 body: "Good news! Build #${env.BUILD_NUMBER} of ${env.JOB_NAME} completed successfully.\n\nCheck it out: ${env.BUILD_URL}"
        }
        failure {
            mail to: 'suzannedsouza100@gmail.com',
                 subject: "FAILED: Build #${env.BUILD_NUMBER} - ${env.JOB_NAME}",
                 body: "Build #${env.BUILD_NUMBER} of ${env.JOB_NAME} failed.\n\nCheck logs: ${env.BUILD_URL}console"
        }
    }
}
