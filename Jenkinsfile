pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('test-token')  // Replace with your Jenkins credentials ID
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the code from your Git repository
                checkout scm
            }
        }

        stage('Build') {
            steps {
                // Build your project
                script {
                    sh 'mvn clean install'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                // Set the environment for SonarQube analysis and run SonarScanner
                sh '''
			        /var/jenkins_home/tools/hudson.plugins.sonar.SonarRunnerInstallation/SonarScanner/bin/sonar-scanner
			    '''
            }
        }

        stage('Quality Gate') {
            steps {
                // Wait for SonarQube analysis to complete and check the quality gate status
                script {
                    def qualityGate = waitForQualityGate()
                    if (qualityGate.status != 'OK') {
                        error "Quality gate failed: ${qualityGate.status}"
                    }
                }
            }
        }
    }
}
