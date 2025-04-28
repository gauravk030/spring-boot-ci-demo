pipeline {
    agent any

    tools {
        maven 'Maven 3.8.7'  // Ensure this matches your Maven installation
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

        stage('Set Build Name') {
            steps {
                script {
                    // Extract branch name
                    def branchName = env.GIT_BRANCH?.replaceFirst(/^origin\//, '') ?: 'unknown-branch'

                    // Extract Maven version from pom.xml
                    def version = sh(script: "mvn help:evaluate -Dexpression=project.version -q -DforceStdout", returnStdout: true).trim()

                    // Set Jenkins build display name
                    currentBuild.displayName = "${branchName} - ${version} #${BUILD_NUMBER}"

                    echo "Build Display Name set to: ${currentBuild.displayName}"
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

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${env.SONARQUBE_SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectKey=spring-boot-ci-demo -Dsonar.sources=src -Dsonar.java.binaries=target"
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

    post {
        success {
            echo 'Build and analysis were successful!'
        }
        failure {
            echo 'Build or analysis failed!'
        }
    }
}
