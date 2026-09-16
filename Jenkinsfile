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
    }

    post {
        always {
            junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
        }
    }
}
