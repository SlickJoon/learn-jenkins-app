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
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
        stage("Deploy Staging") {
            agent {
                docker {
                    image "node:18-alpine"
                    reuseNode true
                }
            }
            // Get rid of --prod to make it staging
            steps {
                sh '''
                   echo "Hello World!!"
                   npm install netlify-cli@20.1.1 node-jq
                   node_modules/.bin/netlify --version
                   echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                   node_modules/.bin/netlify status
                   node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                   
                '''
                script {
                    env.STAGING_URL = sh(script: "node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json", returnStdout: true)
                }
            }
            
        }
        stage("Staging E2E") {
            // end to end test using playwright tool
            agent {
                docker {
                    image "mcr.microsoft.com/playwright:v1.39.0-jammy"
                    reuseNode true
                    // args '-u root:root'
                }
            }
            environment {
                CI_ENVIRONMENT_URL = "${env.STAGING_URL}"
            }
            steps {
                sh '''
                echo "Hello World"
                npx playwright test --reporter=html
                '''         
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
        stage('Approval') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Do you wish to deploy to production? ', ok: 'Yes, I am sure!'
                }
             }
        }
        // stage("Deploy Prod") {
        //     agent {
        //         docker {
        //             image "node:18-alpine"
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //         sh '''
        //            echo "Hello World!!"
        //            npm install netlify-cli@20.1.1
        //            node_modules/.bin/netlify --version
        //            echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
        //            node_modules/.bin/netlify status
        //            node_modules/.bin/netlify deploy --dir=build --prod
        //         '''
                
        //     }
        // }
        stage("Deploy Prod") {
            // end to end test using playwright tool
            agent {
                docker {
                    image "mcr.microsoft.com/playwright:v1.39.0-jammy"
                    reuseNode true
                    // args '-u root:root'
                }
            }
            environment {
                CI_ENVIRONMENT_URL = 'https://euphonious-sopapillas-7284e1.netlify.app'
            }
            steps {
                sh '''

                echo "Hello World!!"
                node --version
                npm install netlify-cli@20.1.1
                node_modules/.bin/netlify --version
                echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --dir=build --prod
                npx playwright test --reporter=html
                '''         
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
    
}