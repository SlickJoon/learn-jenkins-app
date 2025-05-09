pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '544c48c7-8c6d-4651-852a-abb849bba628'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        stage("Build") {
            agent {
                docker {
                    image "node:18-alpine"
                    reuseNode true
                }
            }
            steps {
                sh '''
                   echo "Hello World"
                   ls -la
                   node --version
                   npm --version
                   npm ci
                   npm run build
                   ls -la
                '''
                
            }
        }
        stage('Tests') {
            parallel {
                stage("Unit Test") {
                    agent {
                        docker {
                            image "node:18-alpine"
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                        echo "Hello World"
                        ls -la
                        #test -f build/index.html
                        npm test
                        '''         
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }
                stage("E2E") {
                    // end to end test using playwright tool
                    agent {
                        docker {
                            image "mcr.microsoft.com/playwright:v1.39.0-jammy"
                            reuseNode true
                            // args '-u root:root'
                        }
                    }
                    steps {
                        sh '''
                        echo "Hello World"
                        npm install serve
                        node_modules/.bin/serve -s build &
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
        stage("Deploy") {
            agent {
                docker {
                    image "node:18-alpine"
                    reuseNode true
                }
            }
            steps {
                sh '''
                   echo "Hello World!!"
                   npm install netlify-cli@20.1.1
                   node_modules/.bin/netlify --version
                   echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                   node_modules/.bin/netlify status
                   node_modules/.bin/netlify deploy --dir=build --prod
                '''
                
            }
        }
    }
    
}