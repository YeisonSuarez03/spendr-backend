pipeline {
    agent any

    environment {
        // Docker and Git configuration
        DOCKER_COMPOSE_FILE = 'docker-compose.test.yml'
        
        // Test configuration
        TEST_COMMAND = 'npm test'
        
        // Deployment simulation
        DEPLOY_MESSAGE = 'Deploying app to server...'
    }

    options {
        // Keep last 10 builds
        buildDiscarder(logRotator(numToKeepStr: '10'))
        
        // Add timestamps to console output
        timestamps()
        
        // Timeout after 30 minutes
        timeout(time: 30, unit: 'MINUTES')
        
        // Disable concurrent builds for safety
        disableConcurrentBuilds()
    }

    stages {
        stage('Setup') {
            steps {
                echo '========== Stage: Setup =========='
                echo "Workspace: ${WORKSPACE}"
                echo "Current directory:"
                sh 'pwd && ls -la'
            }
        }

        stage('Initialize Docker') {
            steps {
                echo '========== Stage: Initialize Docker =========='
                echo "Starting Docker containers using ${DOCKER_COMPOSE_FILE}..."
                sh '''
                    cd ${WORKSPACE}
                    echo "Current directory: $(pwd)"
                    echo "Files present:"
                    ls -la | head -20
                    
                    if [ ! -f "${DOCKER_COMPOSE_FILE}" ]; then
                        echo "ERROR: ${DOCKER_COMPOSE_FILE} not found!"
                        exit 1
                    fi
                    
                    echo "Building and starting containers..."
                    docker compose -f ${DOCKER_COMPOSE_FILE} up -d --build
                    
                    echo "Waiting for services to be ready..."
                    sleep 10
                    
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
                    cd ${WORKSPACE}
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
                cd ${WORKSPACE} || echo "Failed to cd to workspace"
                echo "Current directory: $(pwd)"
                
                if [ -f "${DOCKER_COMPOSE_FILE}" ]; then
                    echo "Stopping and removing containers..."
                    docker compose -f ${DOCKER_COMPOSE_FILE} down --remove-orphans || true
                    
                    echo "Cleanup complete"
                    docker compose -f ${DOCKER_COMPOSE_FILE} ps || echo "No containers running"
                else
                    echo "WARNING: ${DOCKER_COMPOSE_FILE} not found in workspace"
                    echo "Cleaning up all spendr-related containers manually..."
                    docker ps -a | grep spendr || echo "No spendr containers found"
                    docker compose down --remove-orphans || true
                fi
            '''
        }

        success {
            echo '========== Build Successful =========='
            echo '✓ Pipeline completed successfully'
        }

        failure {
            echo '========== Build Failed =========='
            echo '✗ Pipeline failed. Check logs above for details.'
        }

        unstable {
            echo '========== Build Unstable =========='
            echo '⚠ Pipeline completed with warnings.'
        }

        cleanup {
            echo '========== Final Cleanup =========='
            deleteDir()
        }
    }
}
