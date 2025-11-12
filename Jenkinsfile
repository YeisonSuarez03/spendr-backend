pipeline {
    agent any

    environment {
        // Docker and Git configuration
        DOCKER_COMPOSE_FILE = 'docker-compose.test.yml'
        GIT_REPO = 'https://github.com/YeisonSuarez03/spendr-backend.git'
        BRANCH = 'master'
        
        // Test configuration
        TEST_COMMAND = 'npm test'
        
        // Deployment simulation
        DEPLOY_MESSAGE = 'Deploying app to server...'
    }

    triggers {
        // Trigger on GitHub push to master branch
        githubPush()
    }

    options {
        // Keep last 10 builds
        buildDiscarder(logRotator(numToKeepStr: '10'))
        
        // Add timestamps to console output
        timestamps()
        
        // Timeout after 30 minutes
        timeout(time: 30, unit: 'MINUTES')
    }

    stages {
        stage('Checkout') {
            steps {
                echo '========== Stage: Checkout =========='
                echo "Checking out code from ${GIT_REPO} (branch: ${BRANCH})"
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "refs/heads/${BRANCH}"]],
                    userRemoteConfigs: [[url: "${GIT_REPO}"]]
                ])
                echo 'Code checked out successfully'
            }
        }

        stage('Initialize Docker') {
            steps {
                echo '========== Stage: Initialize Docker =========='
                echo "Starting Docker containers using ${DOCKER_COMPOSE_FILE}..."
                sh '''
                    echo "Building and starting containers..."
                    docker compose -f ${DOCKER_COMPOSE_FILE} up -d --build
                    
                    echo "Waiting for services to be ready..."
                    sleep 5
                    
                    echo "Current running containers:"
                    docker compose -f ${DOCKER_COMPOSE_FILE} ps
                '''
                echo 'Docker containers initialized successfully'
            }
        }

        stage('Execute Tests') {
            steps {
                echo '========== Stage: Execute Tests =========='
                echo 'Running Jest tests inside test-runner container...'
                sh '''
                    echo "Running tests..."
                    docker compose -f ${DOCKER_COMPOSE_FILE} run --rm test-runner ${TEST_COMMAND}
                    
                    if [ $? -eq 0 ]; then
                        echo "✓ All tests passed successfully!"
                    else
                        echo "✗ Tests failed!"
                        exit 1
                    fi
                '''
            }
        }

        stage('Deploy') {
            when {
                // Only deploy if on master branch and previous stages succeeded
                branch 'master'
                // Add condition to only deploy on successful build
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                echo '========== Stage: Deploy =========='
                echo "${DEPLOY_MESSAGE}"
                sh '''
                    echo "Simulating deployment to production server..."
                    echo "Deploying app..."
                    sleep 2
                    echo "✓ Deployment simulation complete!"
                '''
            }
        }
    }

    post {
        always {
            echo '========== Post: Cleanup =========='
            echo 'Bringing down all Docker containers...'
            sh '''
                echo "Stopping and removing containers..."
                docker compose -f ${DOCKER_COMPOSE_FILE} down --remove-orphans || true
                
                echo "Cleanup complete"
                docker compose -f ${DOCKER_COMPOSE_FILE} ps || echo "No containers running"
            '''
        }

        success {
            echo '========== Build Successful =========='
            echo '✓ Pipeline completed successfully'
            // Optional: Add notifications (email, Slack, etc.)
        }

        failure {
            echo '========== Build Failed =========='
            echo '✗ Pipeline failed. Check logs above for details.'
            // Optional: Add notifications (email, Slack, etc.)
        }

        unstable {
            echo '========== Build Unstable =========='
            echo '⚠ Pipeline completed with warnings.'
        }
    }
}
