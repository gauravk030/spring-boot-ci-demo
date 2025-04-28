pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('sonar-key') // Jenkins credential ID
    }

    tools {
        maven 'Maven 3.8.7' // Or whatever Maven you installed
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQubeLocal') {
                    sh '''
                        mvn sonar:sonar \
                          -Dsonar.projectKey=spring-boot-ci-demo \
                          -Dsonar.projectName="spring-boot-ci-demo" \
                          -Dsonar.host.url=http://localhost:9000 \
                          -Dsonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}
