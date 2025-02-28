pipeline {
    agent { node { label 'built-in' } }

    environment{
        NETLIFY_SITE_ID = '6ebddb7e-089d-4369-8c93-009010169d86'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.2.${BUILD_ID}"
    }

    stages {
        stage('AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    args '--entrypoint=''"
                }
            }
            steps {
                sh '''
                    aws --version
                '''
            }
        }

        stage('Build') {
            agent { node { label 'built-in' } }
            steps {
                sh '''
                    echo "Small change"
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        
        stage('Run Test'){
            parallel {
                stage('Unit Test') {
                    agent { node { label 'built-in' } }
                    steps {
                        sh '''
                            npm install
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }
                stage('E2E Test') {
                    agent { node { label 'built-in' } }
                    steps {
                        sh '''
                            serve -s build &
                            sleep 5
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy staging') {
            agent { node { label 'built-in' } }
            environment{
                CI_ENVIRONMENT_URL = 'STAGING_URL_TO_BE_SET'
            }
            steps {
                sh '''
                    netlify --version
                    echo "Deploying to staging . Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(jq -r '.deploy_url' deploy-output.json)
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
        
        stage('Deploy prod') {
            agent { node { label 'built-in' } }
            environment{
                CI_ENVIRONMENT_URL = 'https://magenta-conkies-a9ebf2.netlify.app'
            }
            steps {
                sh '''
                    node --version
                    netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --prod
                    npx playwright test --reporter=html
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
