pipeline {

    environment {
        NETLIFY_SITE_ID = '38bd6e34-3979-48fe-b09f-399cf2ec03bb'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

    agent any

    stages {

        stage('Docker') {
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
                            image 'my-playwright'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            echo "Local E2E testing stage"
                            serve -s build &
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
                            image 'my-playwright'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            node --version
                            netlify --version
                            echo "Deploying to Staging. Site ID: $NETLIFY_SITE_ID"
                            netlify status
                            netlify deploy --dir=build --json > deploy-output.json
                            sleep 10
                            CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy-output.json)
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
        stage('Deploy Prod') {
                    environment {
                        CI_ENVIRONMENT_URL = 'https://stellular-liger-3b235c.netlify.app'
                    }
                    agent {
                        docker {
                            image 'my-playwright'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            node --version
                            netlify --version
                            echo "Deploying to Prod. Site ID: $NETLIFY_SITE_ID"
                            netlify status
                            netlify deploy --dir=build --prod
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