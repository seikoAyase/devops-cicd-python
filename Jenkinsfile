
pipeline {
    agent any

    environment {
        DOCKER_IMAGE_NAME = 'python-microservice-cicd'
        TEST_RESULTS_DIR = 'test-results'
    }

    options {
        timestamps()
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh 'docker build -t python-microservice-cicd:${BUILD_NUMBER} .'
            }
        }

        stage('Run Tests in Docker') {
            steps {
                script {
                    def containerName = "assignment5-tests-${env.BUILD_NUMBER}"
                    def imageName = "${env.DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}"

                    sh "docker rm -f ${containerName} 2>/dev/null || true"

                    def testStatus = sh(
                        script: """
                            docker run --name ${containerName} ${imageName} \
                            sh -c 'mkdir -p test-results && python -m pytest tests/ -v \
                            --cov=app \
                            --cov-report=xml:test-results/coverage.xml \
                            --cov-report=term-missing \
                            --junitxml=test-results/junit-report.xml \
                            --html=test-results/pytest-report.html'
                        """,
                        returnStatus: true
                    )

                    sh "mkdir -p ${TEST_RESULTS_DIR}"

                    def copyStatus = sh(
                        script: "docker cp ${containerName}:/usr/src/app/test-results/. ${TEST_RESULTS_DIR}/",
                        returnStatus: true
                    )

                    sh "docker rm -f ${containerName}"

                    if (copyStatus != 0) {
                        error('Failed to copy test reports.')
                    }

                    if (testStatus != 0) {
                        error("Tests failed with exit code ${testStatus}")
                    }
                }
            }
        }

        stage('Archive & Display Results') {
            steps {
                echo 'Publishing test results...'

                junit 'test-results/junit-report.xml'

                archiveArtifacts(
                    artifacts: 'test-results/*.xml,test-results/*.html',
                    fingerprint: true
                )
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution finished.'
        }

        success {
            echo 'All stages completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check Console Output.'
        }
    }
}
