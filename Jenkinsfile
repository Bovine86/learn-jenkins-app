pipeline {

    environment {
        NETLIFY_SITE_ID = '38bd6e34-3979-48fe-b09f-399cf2ec03bb'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    agent any

    stages {
        /*
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
        */
        stage('Tests') {
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
                            echo "Test stage"
                            if [ ! -f build/index.html ]; then
                                exit 1
                            fi
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'test1-results/junit.xml'
                        }
                    }
                }
                stage('E2E Tests') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            echo "E2E testing stage"
                            if [ ! -L node_modules/.bin/serve ]; then
                                npm install serve
                            fi
                            node_modules/.bin/serve -s build &
                            sleep 10
                            #npx playwright test
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
        stage('Deploy') {
                agent {
                    docker {
                        image 'node:18-alpine'
                        reuseNode true
                    }
                }
                steps {
                    sh '''
                        echo "Deployment stage"
                        if [ ! -L node_modules/.bin/netlify ]; then
                            npm install netlify-cli@20.1.1
                        fi
                        node_modules/.bin/netlify --version
                        echo "Deploying to Production. Site ID: $NETLIFY_SITE_ID"
                        node_modules/.bin/netlify status
                    '''
                }
            }
    }
}