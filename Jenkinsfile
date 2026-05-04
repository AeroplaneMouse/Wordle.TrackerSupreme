pipeline {
    agent any

    environment {
        // These are typically already available in Jenkins, but we make it explicit
        COMMIT_SHA = "${env.GIT_COMMIT}"
        BUILD_NUM  = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Set image tag') {
            steps {
                script {
                    env.IMAGE_TAG = env.GIT_COMMIT.take(7)
                    echo "Using IMAGE_TAG=${env.IMAGE_TAG}"
                }
            }
        }

        stage('Build Docker images') {
            steps {
                script {
                    withEnv(["IMAGE_TAG=${env.IMAGE_TAG}"]) {
                        withEnv(["DOCKER_BUILDKIT=1"]) {
                            sh """
                                docker compose -f compose.build.yml \
                                --profile build-app build \
                                --build-arg COMMIT_SHA=${COMMIT_SHA} \
                                --build-arg BUILD_NUMBER=${BUILD_NUM}
                            """
                        }
                    }
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                sh """
                    docker compose -f compose.build.yml push web api migrator
                """
            }
        }
    }

    post {
        success {
            echo 'Docker images built successfully.'
        }
        failure {
            echo 'Build failed.'
        }
    }
}