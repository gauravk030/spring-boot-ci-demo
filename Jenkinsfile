pipeline {
    agent any

    tools {
        maven 'Maven 3.6.3'  // Update with the correct Maven version name
        sonarScanner 'sonar test'  // Make sure this matches the name of your SonarQube scanner setup
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/gauravk030/spring-boot-ci-demo.git'  // GitHub repository URL
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
            echo 'Build, analysis, and deployment were successful!'
        }
        failure {
            echo 'There was an issue with the build or deployment.'
        }
    }
}
