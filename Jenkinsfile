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

        stage('Build Docker Images') {
            parallel {
                stage('Build API') {
                    steps {
                        script {
                            sh """
                                docker compose -f compose.build.yml \
                                --profile build-app build api \
                                --build-arg COMMIT_SHA=${COMMIT_SHA} \
                                --build-arg BUILD_NUMBER=${BUILD_NUM}
                            """
                        }
                    }
                }

                stage('Build Migrator') {
                    steps {
                        script {
                            sh """
                                docker compose -f compose.build.yml \
                                --profile build-app build migrator \
                                --build-arg COMMIT_SHA=${COMMIT_SHA} \
                                --build-arg BUILD_NUMBER=${BUILD_NUM}
                            """
                        }
                    }
                }

                stage('Build Web') {
                    steps {
                        script {
                            sh """
                                docker compose -f compose.build.yml \
                                --profile build-app build web \
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