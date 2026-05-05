pipeline {
    agent any

    parameters {
        string(name: 'SHORT_HASH', description: 'Short commit hash (e.g. abc1234)')
        choice(name: 'TARGET_ENV', choices: ['qa', 'prod'], description: 'Deployment target')
    }

    environment {
        REGISTRY = "registry.trackersupreme.dk"
    }

    stages {
        stage('Validate Input') {
            steps {
                script {
                    if (!params.SHORT_HASH?.trim()) {
                        error("SHORT_HASH is required")
                    }

                    env.TARGET_TAG = params.TARGET_ENV == 'qa' ? 'qa-latest' : 'latest'

                    echo "Promoting images:"
                    echo "  From: ${params.SHORT_HASH}"
                    echo "  To:   ${env.TARGET_TAG}"
                }
            }
        }

        stage('Pull Images') {
            steps {
                sh """
                    docker pull ${REGISTRY}/web:${params.SHORT_HASH}
                    docker pull ${REGISTRY}/api:${params.SHORT_HASH}
                    docker pull ${REGISTRY}/migrator:${params.SHORT_HASH}
                """
            }
        }

        stage('Tag Images') {
            steps {
                sh """
                    docker tag ${REGISTRY}/web:${params.SHORT_HASH} ${REGISTRY}/web:${TARGET_TAG}
                    docker tag ${REGISTRY}/api:${params.SHORT_HASH} ${REGISTRY}/api:${TARGET_TAG}
                    docker tag ${REGISTRY}/migrator:${params.SHORT_HASH} ${REGISTRY}/migrator:${TARGET_TAG}
                """
            }
        }

        stage('Push Promoted Tags') {
            steps {
                sh """
                    docker push ${REGISTRY}/web:${TARGET_TAG}
                    docker push ${REGISTRY}/api:${TARGET_TAG}
                    docker push ${REGISTRY}/migrator:${TARGET_TAG}
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