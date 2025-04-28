pipeline {
    agent any

    tools {
        maven 'Maven 3.8.7'  // Ensure this matches your Maven installation
        // Use SonarRunnerInstallation instead of sonarScanner
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/gauravk030/spring-boot-ci-demo.git'  // Replace with your repository
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
