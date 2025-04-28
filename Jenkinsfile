pipeline {
    agent any

    tools {
        sonarQubeScanner 'SonarQubeScanner'  // Name of the SonarQube Scanner configured above
    }
    
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/gauravk030/spring-boot-ci-demo.git'
            }
        }
        stage('Build') {
            steps {
                sh './mvnw clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('SonarQube') {  // Ensure 'SonarQube' is your SonarQube server name
                        sh 'mvn clean verify sonar:sonar'
                    }
                }
            }
        }
    }
}

