pipeline {
    agent any

    parameters {
        string(name: 'SHORT_HASH', description: 'Commit hash (e.g. abc1234)')
        choice(name: 'TARGET_ENV', choices: ['QA', 'PROD'], description: 'Deployment target')
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

                    env.TARGET_TAG = params.TARGET_ENV == 'QA' ? 'qa-latest' : 'latest'

                    echo "Promoting images:"
                    echo "  From: ${params.SHORT_HASH}"
                    echo "  To:   ${env.TARGET_TAG}"
                }
            }
        }

        stage('Pull Images') {
            steps {
                sh """
                    docker pull ${REGISTRY}/wordle-trackersupreme-web:${params.SHORT_HASH}
                    docker pull ${REGISTRY}/wordle-trackersupreme-api:${params.SHORT_HASH}
                    docker pull ${REGISTRY}/wordle-trackersupreme-migrator:${params.SHORT_HASH}
                """
            }
        }

        stage('Tag Images') {
            steps {
                sh """
                    docker tag ${REGISTRY}/wordle-trackersupreme-web:${params.SHORT_HASH} ${REGISTRY}/wordle-trackersupreme-web:${TARGET_TAG}
                    docker tag ${REGISTRY}/wordle-trackersupreme-api:${params.SHORT_HASH} ${REGISTRY}/wordle-trackersupreme-api:${TARGET_TAG}
                    docker tag ${REGISTRY}/wordle-trackersupreme-migrator:${params.SHORT_HASH} ${REGISTRY}/wordle-trackersupreme-migrator:${TARGET_TAG}
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