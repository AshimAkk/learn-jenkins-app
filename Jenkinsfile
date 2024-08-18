pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'c0d35b04-6b30-4918-9973-c642ac725cd2'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')

    }

    stages {
        
        stage('Tests') {
            parallel {
                stage('Unit Test') {
              agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
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
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            steps {
                sh '''
                npm install serve
                node_modules/.bin/serve -s build &
                sleep 10
                npx playwright test --reporter=line
                '''
               
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
                    npm install netlify-cli 
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                    echo "message"

                '''
            }
        }
        
        stage('Prod E2E') {
              agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

              environment {
                    CI_ENVIRONMENT_URL = 'https://spectacular-gumdrop-1fa3b7.netlify.app'
        
    }
            steps {
                sh '''
                npx playwright test --reporter=line
                '''
               
            }
        }
    }

    post {
        always{
            junit 'jest-results/junit.xml'
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwrite HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
    }
}
