pipeline {
    agent any

    tools {
        maven 'Maven 3.8.7'
    }

    environment {
        SONARQUBE_SCANNER_HOME = tool 'SonarScanner'
        DOCKER_REGISTRY = 'gauravkhrnr'        // Your DockerHub username
        IMAGE_NAME = 'spring-boot-ci-demo'
        KUBE_NAMESPACE = 'default'             // Kubernetes namespace
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Set Build Name') {
            steps {
                script {
                    sh 'mvn validate'

                    def branchName = env.GIT_BRANCH?.replaceFirst(/^origin\//, '') ?: 'master'
                    def version = sh(
                        script: "mvn help:evaluate -Dexpression=project.version -q -DforceStdout",
                        returnStdout: true
                    ).trim()

                    version = version.replaceAll('[^\\x20-\\x7E]', '')
                    if (!version) {
                        version = 'unknown-version'
                    }
                    currentBuild.displayName = "${branchName} - ${version} #${BUILD_NUMBER}"
                    env.APP_VERSION = version
                    echo "✅ Build Name: ${currentBuild.displayName}"
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQubeServer') {
                    sh "${SONARQUBE_SCANNER_HOME}/bin/sonar-scanner"
                }
            }
        }

        stage('Build and Unit Test') {
            steps {
                sh 'mvn clean verify'
            }
        }

        stage('Docker Build and Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    script {
                        def fullImageName = "${DOCKER_USERNAME}/${IMAGE_NAME}:${APP_VERSION}"
                        def latestImageName = "${DOCKER_USERNAME}/${IMAGE_NAME}:latest"
                        
                        echo "📦 Building Docker Images: ${fullImageName} and ${latestImageName}"

                        sh """
                            docker build -t "${fullImageName}" -t "${latestImageName}" .
                            echo "${DOCKER_PASSWORD}" | docker login -u "${DOCKER_USERNAME}" --password-stdin
                            docker push "${fullImageName}"
                            docker push "${latestImageName}"
                            docker logout
                        """

                        env.FULL_IMAGE_NAME = fullImageName
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo "🚀 Deploying '${FULL_IMAGE_NAME}' to Kubernetes namespace '${KUBE_NAMESPACE}'..."
                    sh """
                        kubectl set image deployment/${IMAGE_NAME} ${IMAGE_NAME}=${FULL_IMAGE_NAME} --namespace=${KUBE_NAMESPACE}
                        kubectl rollout status deployment/${IMAGE_NAME} --namespace=${KUBE_NAMESPACE}
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed! Please check the console logs for errors!'
        }
        always {
            cleanWs()
        }
    }
}
