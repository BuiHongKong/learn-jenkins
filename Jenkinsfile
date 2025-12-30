pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node -v
                    npm -v
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm test
                    cat test-results/junit.xml || echo "Test results file not found"
                '''
            }
        }
        stage('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install serve
                    npx serve -s build -l 3000 &
                    SERVE_PID=$!
                    echo "Waiting for server to start..."
                    sleep 5
                    until curl -f http://localhost:3000 > /dev/null 2>&1; do
                        echo "Waiting for server..."
                        sleep 1
                    done
                    echo "Server is ready!"
                    npx playwright test
                    kill $SERVE_PID || true
                '''
            }
            post {
                always {
                    sh 'pkill -f "serve.*build" || true'
                    junit 'test-results/junit.xml'
                }
            }
        }
    }

    post {
        always {
            junit 'jest-results/junit.xml'
        }
    }
}