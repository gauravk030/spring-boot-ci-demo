pipeline {
    agent any

    tools {
        maven 'Maven 3.8.7'  // Ensure this matches your Maven installation
    }
    environment {
        SONARQUBE_SCANNER_HOME = tool 'SonarScanner'   // SonarScanner installation name
    }
    stages {
        stage('Checkout') {
            steps {
                setGitHubPullRequestStatus context: 'CI/CD', status: 'PENDING', description: 'Checking out project'
                git branch: 'master', url: 'https://github.com/gauravk030/spring-boot-ci-demo.git'
                setGitHubPullRequestStatus context: 'CI/CD', status: 'SUCCESS', description: 'Checkout completed'
            }
        }

        stage('Build') {
            steps {
                script {
                    setGitHubPullRequestStatus context: 'CI/CD', status: 'PENDING', description: 'Building project'
                    sh 'mvn clean install'
                    setGitHubPullRequestStatus context: 'CI/CD', status: 'SUCCESS', description: 'Build completed'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    setGitHubPullRequestStatus context: 'CI/CD', status: 'PENDING', description: 'Running SonarQube analysis'
                    sh "${env.SONARQUBE_SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectKey=spring-boot-ci-demo -Dsonar.sources=src -Dsonar.java.binaries=target"
                    setGitHubPullRequestStatus context: 'CI/CD', status: 'SUCCESS', description: 'SonarQube analysis completed'
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    setGitHubPullRequestStatus context: 'CI/CD', status: 'PENDING', description: 'Waiting for Quality Gate result'
                    waitForQualityGate abortPipeline: true
                    setGitHubPullRequestStatus context: 'CI/CD', status: 'SUCCESS', description: 'Quality Gate passed'
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
