pipeline {
    agent any

    tools {
        maven 'Maven 3.8.7'
    }
    environment {
        SONARQUBE_SCANNER_HOME = tool 'SonarScanner'
    }
    stages {
        stage('Checkout') {
            steps {
                githubNotify context: 'CI/CD', state: 'PENDING', description: 'Checking out code...', credentialsId: 'git-token'
                git branch: 'master', url: 'https://github.com/gauravk030/spring-boot-ci-demo.git'
                githubNotify context: 'CI/CD', state: 'SUCCESS', description: 'Checkout completed', credentialsId: 'git-token'
            }
        }

        stage('Build') {
            steps {
                script {
                    githubNotify context: 'CI/CD', state: 'PENDING', description: 'Building project...', credentialsId: 'git-token'
                    sh 'mvn clean install'
                    githubNotify context: 'CI/CD', state: 'SUCCESS', description: 'Build successful', credentialsId: 'git-token'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    githubNotify context: 'CI/CD', state: 'PENDING', description: 'Running SonarQube analysis...', credentialsId: 'git-token'
                    sh "${env.SONARQUBE_SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectKey=spring-boot-ci-demo -Dsonar.sources=src -Dsonar.java.binaries=target"
                    githubNotify context: 'CI/CD', state: 'SUCCESS', description: 'SonarQube analysis completed', credentialsId: 'git-token'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    githubNotify context: 'CI/CD', state: 'PENDING', description: 'Waiting for Quality Gate...', credentialsId: 'git-token'
                    waitForQualityGate abortPipeline: true
                    githubNotify context: 'CI/CD', state: 'SUCCESS', description: 'Quality Gate passed', credentialsId: 'git-token'
                }
            }
        }
    }
    post {
        success {
            echo 'Build and analysis were successful!'
        }
        failure {
            echo 'Build or analysis failed!'
        }
    }
}
