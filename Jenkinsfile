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
                setGitHubPullRequestStatus state: 'PENDING', context: 'CI/CD'
                git branch: 'master', url: 'https://github.com/gauravk030/spring-boot-ci-demo.git'
                setGitHubPullRequestStatus state: 'SUCCESS', context: 'CI/CD'
            }
        }

        stage('Build') {
            steps {
                script {
                    setGitHubPullRequestStatus state: 'PENDING', context: 'CI/CD'
                    sh 'mvn clean install'
                    setGitHubPullRequestStatus state: 'SUCCESS', context: 'CI/CD'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    setGitHubPullRequestStatus state: 'PENDING', context: 'CI/CD'
                    sh "${env.SONARQUBE_SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectKey=spring-boot-ci-demo -Dsonar.sources=src -Dsonar.java.binaries=target"
                    setGitHubPullRequestStatus state: 'SUCCESS', context: 'CI/CD'
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    setGitHubPullRequestStatus state: 'PENDING', context: 'CI/CD'
                    waitForQualityGate abortPipeline: true
                    setGitHubPullRequestStatus state: 'SUCCESS', context: 'CI/CD'
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
