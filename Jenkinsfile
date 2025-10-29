pipeline {
    agent any
    environment {
        // Set the new path for the current Jenkins job execution
        PATH = "$HOME/.npm-global/bin:$PATH" 
        NETLIFY_SITE_ID = '2c7363ca-155f-4c2a-93f3-7c0d04af22c7'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

    stages {
    
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
        
        stage('Tests') {
            parallel {
                stage('Unit Test') {
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
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
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
                            npm install serve
                            npx serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

    
        stage('Deploy Staging') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.56.1-jammy'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = "STAGING_URL_TO_BE_SET"
            }

            steps {
                sh '''
                    npm install node-jq netlify-cli@20.1.1 -g --prefix=$HOME/.npm-global 
                    export PATH=$HOME/.npm-global/bin:$PATH
                    netlify --version
                    echo "Deploying to Netlify site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build  --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy-output.json)
                    npx playwright test  --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        stage('Depoly Prod') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.56.1-jammy'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://splendid-cannoli-d120e2.netlify.app'
            }

            steps {
                sh '''
                    node --version
                    npm install netlify-cli@20.1.1 -g --prefix=$HOME/.npm-global
                    export PATH=$HOME/.npm-global/bin:$PATH
                    netlify --version
                    echo "Deploying to Netlify site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --prod
                    npx playwright test  --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }


    }
}
