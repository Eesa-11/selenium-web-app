pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = "orakzaieesa11/selenium-web-app"
        CONTAINER_NAME = "selenium-web-app"
        APP_PORT = "5000"
    }
    
    stages {
        stage('Cleanup') {
            steps {
                script {
                    sh '''
                        docker stop ${CONTAINER_NAME} || true
                        docker rm ${CONTAINER_NAME} || true
                        docker rmi ${DOCKER_IMAGE}:latest || true
                    '''
                }
            }
        }
        
        stage('Build') {
            steps {
                script {
                    sh 'docker build -t ${DOCKER_IMAGE}:latest .'
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    sh '''
                        docker run -d \
                            --name ${CONTAINER_NAME} \
                            -p ${APP_PORT}:5000 \
                            ${DOCKER_IMAGE}:latest
                    '''
                }
            }
        }
        
        stage('Wait for App') {
            steps {
                script {
                    sh 'sleep 10'
                }
            }
        }
        
        stage('Test') {
            steps {
                script {
                    sh '''
                        # Clone test repository
                        rm -rf selenium-test-cases
                        git clone https://github.com/Eesa-11/selenium-test-cases.git
                        cd selenium-test-cases
                        
                        # Run tests in Docker
                        docker run --rm \
                            --network host \
                            -v $(pwd):/tests \
                            -w /tests \
                            -e APP_URL=http://localhost:5000 \
                            markhobson/maven-chrome \
                            /bin/bash -c "pip install -r requirements.txt && python -m pytest test_selenium.py -v --tb=short > test_results.txt 2>&1 || true"
                        
                        # Copy results
                        cat test_results.txt
                    '''
                }
            }
        }
    }
    
    post {
        always {
            script {
                def testResults = sh(
                    script: 'cat selenium-test-cases/test_results.txt || echo "No test results found"',
                    returnStdout: true
                ).trim()
                
                def commitAuthor = sh(
                    script: 'git log -1 --pretty=format:"%ae"',
                    returnStdout: true
                ).trim()
                
                emailext(
                    to: "${commitAuthor}",
                    subject: "Jenkins Build ${currentBuild.currentResult}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: """
                        Build Status: ${currentBuild.currentResult}
                        Job: ${env.JOB_NAME}
                        Build Number: ${env.BUILD_NUMBER}
                        
                        Test Results:
                        ${testResults}
                        
                        Check console output at: ${env.BUILD_URL}
                    """,
                    mimeType: 'text/plain'
                )
            }
        }
        
        failure {
            script {
                sh '''
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true
                '''
            }
        }
    }
}
