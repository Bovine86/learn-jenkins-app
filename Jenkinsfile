pipeline {

    environment {
        NETLIFY_SITE_ID = '38bd6e34-3979-48fe-b09f-399cf2ec03bb'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

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
                    date > timestamp.txt
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        
        stage('Local Tests') {
            parallel {
                stage('Local Unit Tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            echo "Local Unit testing stage"
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'test1-results/junit.xml'
                        }
                    }
                }
                stage('Local E2E Tests') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            echo "Local E2E testing stage"
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            #npx playwright test
                            npx playwright test --reporter=html
                            mkdir -p playwright-report-local
                            cp playwright-report/index.html playwright-report-local/
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report-local', reportFiles: 'index.html', reportName: 'Local E2E', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
        stage('Deploy Staging') {
                    environment {
                        CI_ENVIRONMENT_URL = ""
                    }
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            node --version
                            npm install netlify-cli@20.1.1 node-jq
                            node_modules/.bin/netlify --version
                            echo "Deploying to Staging. Site ID: $NETLIFY_SITE_ID"
                            node_modules/.bin/netlify status
                            node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                            sleep 10
                            CI_ENVIRONMENT_URL=$(node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json)
                            npx playwright test --reporter=html
                            mkdir -p playwright-report-staging
                            cp playwright-report/index.html playwright-report-staging/
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report-staging', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
        stage('Manual Approval') {
            steps {
                echo 'Wating for manual approval...'
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Deploy to production?', ok: "Proceed", cancel: "Abort"
                        }
                }
            } 
        stage('Deploy Prod') {
                    environment {
                        CI_ENVIRONMENT_URL = 'https://stellular-liger-3b235c.netlify.app'
                    }
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            node --version
                            npm install netlify-cli@20.1.1 node-jq
                            node_modules/.bin/netlify --version
                            echo "Deploying to Prod. Site ID: $NETLIFY_SITE_ID"
                            node_modules/.bin/netlify status
                            node_modules/.bin/netlify deploy --dir=build --prod
                            sleep 10
                            npx playwright test --reporter=html
                            mkdir -p playwright-report-prod
                            cp playwright-report/index.html playwright-report-prod/
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report-prod', reportFiles: 'index.html', reportName: 'Prod E2E', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
    }
}