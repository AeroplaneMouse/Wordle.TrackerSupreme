pipeline {
    agent any

    parameters {
        string(name: 'IMAGE_TAG', description: 'Docker image tag to promote')
        choice(name: 'TARGET_ENV', choices: ['QA', 'PROD'], description: 'Deployment target')
    }

    environment {
        REGISTRY = "registry.trackersupreme.dk"
    }

    stages {
        stage('Validate Input') {
            steps {
                script {
                    if (!params.IMAGE_TAG?.trim()) {
                        error("IMAGE_TAG is required")
                    }

                    env.TARGET_TAG = params.TARGET_ENV == 'QA' ? 'qa-latest' : 'latest'

                    echo "Promoting images:"
                    echo "  From: ${params.IMAGE_TAG}"
                    echo "  To:   ${env.TARGET_TAG}"
                }
                script {
                    def IMAGES = [
                        'web',
                        'api',
                        'migrator'
                    ]
                }
            }
        }

        stage('Validate Images Exist') {
            steps {
                script {
                    for (img in IMAGES) {
                        def IMAGE_PATH = "${REGISTRY}/wordle-trackersupreme-${img}:${params.IMAGE_TAG}"
                        echo "Validating image: ${IMAGE_PATH}"
                        sh """
                            docker manifest inspect \
                            ${REGISTRY}/wordle-trackersupreme-${img}:${params.IMAGE_TAG} \
                            > /dev/null
                        """
                    }
                }
            }
        }

        stage('Pull Images') {
            steps {
                sh """
                    docker pull ${REGISTRY}/wordle-trackersupreme-web:${params.IMAGE_TAG}
                    docker pull ${REGISTRY}/wordle-trackersupreme-api:${params.IMAGE_TAG}
                    docker pull ${REGISTRY}/wordle-trackersupreme-migrator:${params.IMAGE_TAG}
                """
            }
        }

        stage('Tag Images') {
            steps {
                sh """
                    docker tag ${REGISTRY}/wordle-trackersupreme-web:${params.IMAGE_TAG} ${REGISTRY}/wordle-trackersupreme-web:${TARGET_TAG}
                    docker tag ${REGISTRY}/wordle-trackersupreme-api:${params.IMAGE_TAG} ${REGISTRY}/wordle-trackersupreme-api:${TARGET_TAG}
                    docker tag ${REGISTRY}/wordle-trackersupreme-migrator:${params.IMAGE_TAG} ${REGISTRY}/wordle-trackersupreme-migrator:${TARGET_TAG}
                """
            }
        }

        stage('Push Promoted Tags') {
            steps {
                sh """
                    docker push ${REGISTRY}/wordle-trackersupreme-web:${TARGET_TAG}
                    docker push ${REGISTRY}/wordle-trackersupreme-api:${TARGET_TAG}
                    docker push ${REGISTRY}/wordle-trackersupreme-migrator:${TARGET_TAG}
                """
            }
        }
    }

    post {
        success {
            echo "Promotion completed"
        }
    }
}