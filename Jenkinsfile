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
                git branch: 'master', url: 'https://github.com/gauravk030/spring-boot-ci-demo.git'  // Use main if that's the default branch
            }
        }

        stage('Build') {
            steps {
                script {
                    // Build the project using Maven
                    sh 'mvn clean install'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${env.SONARQUBE_SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectKey=spring-boot-ci-demo -Dsonar.sources=src -Dsonar.java.binaries=target"
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
