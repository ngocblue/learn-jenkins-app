pipeline {
    agent any
    environment {
        // Set the new path for the current Jenkins job execution
        PATH = "$HOME/.npm-global/bin:$PATH" 
    }

    stages {
        /*
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode  true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                '''

            }
        }
        */
        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode  true
                }
            }
            steps {
                sh '''
                test -f build/index.html
                npm test
                '''
            }


        }

        stage('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.56.1-noble'
                    reuseNode  true
                }
            }
            steps {
                sh '''
                    npm config set prefix "~/.npm-global"
                    npm install -g serve
                    #serve -s build
                    node-modules/.bin/serve -s build  &
                    sleep 10
                    npx playwright test
                '''
            }


        }
    }

    post {
        always {
            junit 'jest-results/junit.xml'
        }
    }
}
