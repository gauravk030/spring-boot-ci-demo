pipeline {
    agent any

    tools {
        maven 'Maven 3.8.7'
    }

    environment {
        SONARQUBE_SCANNER_HOME = tool 'SonarScanner'
        DOCKER_REGISTRY = 'gauravkhrnr'    // Replace with your actual DockerHub username
        IMAGE_NAME = 'spring-boot-ci-demo'
        KUBE_NAMESPACE = 'default'         // Your Kubernetes namespace
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/gauravk030/spring-boot-ci-demo.git'
            }
        }

        stage('Set Build Name') {
            steps {
                script {
                    // Validate Maven and prepare environment before extracting version
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
                    echo "Build Display Name set to: ${currentBuild.displayName}"
                }
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install'
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

        stage('Docker Build and Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    script {
                        // Set the full image name inside the script block
                        def fullImageName = "${DOCKER_USERNAME}/${IMAGE_NAME}:${APP_VERSION}"
                        sh """
                            docker build -t ${fullImageName} .
                            echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                            docker push ${fullImageName}
                        """
                        // Store the image name in an environment variable
                        env.FULL_IMAGE_NAME = fullImageName
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                        kubectl set image deployment/${IMAGE_NAME} ${IMAGE_NAME}=${FULL_IMAGE_NAME} --namespace=${KUBE_NAMESPACE}
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
            echo '❌ Pipeline failed!'
        }
    }
}
