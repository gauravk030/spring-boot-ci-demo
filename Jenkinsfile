pipeline {
    agent any

    tools {
        maven 'Maven 3.8.7'  // Ensure this matches your Maven installation
        // Use SonarRunnerInstallation instead of sonarScanner
       
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/gauravk030/spring-boot-ci-demo.git'  // Use main if that's the default branch
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
