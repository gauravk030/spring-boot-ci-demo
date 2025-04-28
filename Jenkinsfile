pipeline {
    agent any

    tools {
        maven 'Maven 3.6.3'  // Ensure this matches your Maven installation
        // Use SonarRunnerInstallation instead of sonarScanner
        sonarQubeScanner 'SonarQube Scanner'  // Make sure this matches the name of your SonarQube scanner setup in Global Tool Configuration
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/gauravk030/spring-boot-ci-demo.git'  // Replace with your repository
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('SonarQube') {  // Ensure 'SonarQube' matches your SonarQube server configuration
                        sh 'mvn clean verify sonar:sonar'
                    }
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    sh 'mvn clean install'
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
