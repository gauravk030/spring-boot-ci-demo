pipeline {
    agent any

    tools {
        maven 'Maven 3.8.7'
    }

    environment {
        DOCKER_REGISTRY = 'gauravkhrnr'
        IMAGE_NAME = 'spring-boot-ci-demo'
        KUBE_NAMESPACE = 'default'
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

                    def branchName = sh(
                        script: "git rev-parse --abbrev-ref HEAD",
                        returnStdout: true
                    ).trim()

                    def rawVersion = sh(
                        script: "mvn help:evaluate -Dexpression=project.version -q -DforceStdout",
                        returnStdout: true
                    ).trim().replaceAll('[^a-zA-Z0-9._-]', '')

                    if (!rawVersion) {
                        rawVersion = 'unknown-version'
                    }

                    currentBuild.displayName = "${branchName} - ${rawVersion} #${BUILD_NUMBER}"
                    env.APP_VERSION = rawVersion
                    echo "✅ Build Name: ${currentBuild.displayName}"
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

                        echo "📦 Building Docker Images..."

                        sh """
                            docker build -t '${fullImageName}' -t '${latestImageName}' .
                            echo '${DOCKER_PASSWORD}' | docker login -u '${DOCKER_USERNAME}' --password-stdin
                            docker push '${fullImageName}'
                            docker push '${latestImageName}'
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
                    echo "🚀 Deploying '${env.FULL_IMAGE_NAME}' to Kubernetes namespace '${KUBE_NAMESPACE}'..."
                    sh """
                        kubectl set image deployment/${IMAGE_NAME} ${IMAGE_NAME}=${env.FULL_IMAGE_NAME} --namespace=${KUBE_NAMESPACE}
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
