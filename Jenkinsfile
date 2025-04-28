pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/yourname/yourrepo.git'
            }
        }
        stage('Build') {
            steps {
                sh './mvnw clean package'
            }
        }
    }
}
