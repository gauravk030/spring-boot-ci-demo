pipeline {
    agent any

    tools {
        maven 'Maven 3.8.7'  // Ensure this matches your Maven installation
        // Use SonarRunnerInstallation instead of sonarScanner
       
    }
    environment {
        SONARQUBE_SCANNER_HOME = tool 'SonarScanner'   // SonarScanner installation name
    }
    stages {
        stage('Checkout') {
            steps {
              	githubNotify context: 'CI/CD', status: 'PENDING', description: 'Building project', credentialsId: 'git-token'
                git branch: 'master', url: 'https://github.com/gauravk030/spring-boot-ci-demo.git'  // Use main if that's the default branch
            	githubNotify context: 'CI/CD', status: 'SUCCESS', description: 'Build completed', credentialsId: 'git-token'
            }
        }

        stage('Build') {
            steps {
                script {
                	githubNotify context: 'CI/CD', status: 'PENDING', description: 'Building project', credentialsId: 'git-token'
                    // Build the project using Maven
                    sh 'mvn clean install'
                    githubNotify context: 'CI/CD', status: 'SUCCESS', description: 'Build completed', credentialsId: 'git-token'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                	githubNotify context: 'CI/CD', status: 'PENDING', description: 'Building project', credentialsId: 'git-token'
                    sh "${env.SONARQUBE_SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectKey=spring-boot-ci-demo -Dsonar.sources=src -Dsonar.java.binaries=target"
                	githubNotify context: 'CI/CD', status: 'SUCCESS', description: 'Build completed', credentialsId: 'git-token'
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                	githubNotify context: 'CI/CD', status: 'PENDING', description: 'Building project', credentialsId: 'git-token'
                    waitForQualityGate abortPipeline: true
                    githubNotify context: 'CI/CD', status: 'SUCCESS', description: 'Build completed', credentialsId: 'git-token'
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