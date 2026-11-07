
// Jenkins Declarative Pipeline
pipeline {
    agent any // Use 'agent any' if your Jenkins master (built from jenkins/Dockerfile-jenkins) has Docker CLI.
              // This implies the pipeline runs directly on the master or an agent similarly configured.

    environment {
        DOCKER_IMAGE_NAME = "python-microservice-cicd"
        DOCKER_IMAGE_TAG  = "${env.BUILD_ID ?: 'latest'}"
        TEST_RESULTS_DIR  = "test-results" // Define a directory for test outputs
        // CODECOV_TOKEN = credentials('CODECOV_TOKEN_JENKINS')
        // SLACK_WEBHOOK_URL = credentials('SLACK_WEBHOOK_JENKINS')
    }

    options {
        timestamps()
        // buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
                script {
                    // Ensure the test results directory exists and is clean
                    sh "rm -rf ${TEST_RESULTS_DIR}"
                    sh "mkdir -p ${TEST_RESULTS_DIR}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image ${env.DOCKER_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG}..."
                sh "docker build -t ${env.DOCKER_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG} ."
            }
        }

        stage('Run Tests in Docker') {
            steps {
                echo "Running tests inside Docker container..."
                // Mount the Jenkins workspace's TEST_RESULTS_DIR into the container.
                // Pytest inside the container will write its output directly to this mounted volume,
                // which means the files will appear in ${PWD}/${TEST_RESULTS_DIR} on the Jenkins agent.
                sh """
                    docker run --rm \
                        -v "${PWD}/${TEST_RESULTS_DIR}:/usr/src/app/test-results" \
                        -v "${PWD}/app:/usr/src/app/app:ro" \
                        -v "${PWD}/tests:/usr/src/app/tests:ro" \
                        -w /usr/src/app \
                        ${env.DOCKER_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG} \
                        sh -c "pip install pytest-html pytest-cov && \
                               pytest --cov=app --cov-report=xml:test-results/coverage.xml \
                                      --cov-report=term-missing \
                                      --junitxml=test-results/junit-report.xml \
                                      --html=test-results/pytest-report.html tests/"
                """
            }
        }

        stage('Archive & Display Results') {
            steps {
                echo "Archiving test results from ${TEST_RESULTS_DIR}..."
                // JUnit plugin will look for XML files in the specified path relative to the workspace root.
                junit "${TEST_RESULTS_DIR}/junit-report.xml"

                // Publish HTML reports
                publishHTML(target: [
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: "${TEST_RESULTS_DIR}", // Directory containing the HTML report
                    reportFiles: 'pytest-report.html',
                    reportName: 'Pytest HTML Report',
                    // If the HTML report has CSS/JS, you might need to allow serving those:
                    // (This depends on Jenkins Content Security Policy)
                    // allowDangerousSymlinks: true
                ])

                // Archive coverage.xml for potential use with Codecov plugin or manual inspection
                archiveArtifacts artifacts: "${TEST_RESULTS_DIR}/coverage.xml", fingerprint: true
            }
        }

        // Placeholder for Codecov upload using Jenkins plugin or script
        // stage('Upload Coverage to Codecov') {
        //     steps {
        //         echo "Uploading coverage to Codecov..."
        //         // sh "./codecov -t ${CODECOV_TOKEN} -f coverage.xml" // If using Codecov CLI
        //         // Or use Codecov Jenkins Plugin
        //     }
        // }

        // Placeholder for Deployment stage
        // stage('Deploy to Test Server') {
        //     when {
        //         branch 'main' // Only deploy from the main branch
        //         // expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' } // Only if build is successful
        //     }
        //     steps {
        //         echo "Deploying to test server..."
        //         // Add deployment commands here (e.g., Heroku CLI, SSH commands)
        //         // Example: sh "heroku container:push web -a your-heroku-app --arg DOCKER_IMAGE_NAME=${DOCKER_IMAGE_NAME}"
        //         // Example: sh "heroku container:release web -a your-heroku-app"
        //     }
        // }
    }

    post {
        always {
            echo 'Build finished.'
            // Clean up Docker image (optional, to save space on Jenkins server)
            // sh "docker rmi ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} || true"

            // Delete artifacts from workspace to save space
            deleteDir()
        }
        // success {
        //     // Slack Notification for Success
        //     // script {
        //     //     def jobName = env.JOB_NAME
        //     //     def buildNumber = env.BUILD_NUMBER
        //     //     def buildUrl = env.BUILD_URL
        //     //     def message = "✅ Build Successful: ${jobName} #${buildNumber}\n<${buildUrl}|Open Build>"
        //     //     slackSend channel: '#your-ci-channel', message: message, tokenCredentialId: 'SLACK_WEBHOOK_JENKINS'
        //     // }
        // }
        // failure {
        //     // Slack Notification for Failure
        //     // script {
        //     //     def jobName = env.JOB_NAME
        //     //     def buildNumber = env.BUILD_NUMBER
        //     //     def buildUrl = env.BUILD_URL
        //     //     def message = "❌ Build Failed: ${jobName} #${buildNumber}\n<${buildUrl}|Open Build>"
        //     //     slackSend channel: '#your-ci-channel', message: message, tokenCredentialId: 'SLACK_WEBHOOK_JENKINS'
        //     // }
        // }
    }
}
