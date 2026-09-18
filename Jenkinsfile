pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'aec7f25c-75c0-4f3c-b5bc-b01a03cd28f3'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = '1.1.3'
    }

    stages {
        stage ('Docker') {
            steps {
                sh 'docker build -t my-playwright .'
            }
        }

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
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        stage('Run Tests') {
            parallel {
                stage('Unit Tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            test -f ./build/index.html
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                } 
                stage('E2E Tests') {
                    agent {
                        docker {
                            image 'my-playwright'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwritght Local', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                } 
            }
        }
        stage('Deploy staging') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = "STAGING_URL_TO_BE_SET"
            }
            steps {
                sh '''
                    netlify --version
                    echo "Deploying to staging. Project ID: $NETLIFY_PROJECT_ID"
                    netlify status
                    netlify deploy --dir=build --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy-output.json)
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwritght Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
        stage('Deploy prod') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = 'https://effulgent-muffin-35e45b.netlify.app'
            }
            steps {
                sh '''
                    netlify --version
                    echo "Deploying to production. Project ID: $NETLIFY_PROJECT_ID"
                    netlify status
                    netlify deploy --dir=build --prod
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwritght E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}
