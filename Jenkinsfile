pipeline {
    agent any

    stages {
        // stage("Build") {
        //     agent {
        //         docker {
        //             image "node:18-alpine"
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //         sh '''
        //            echo "Hello World"
        //            ls -la
        //            node --version
        //            npm --version
        //            npm ci
        //            npm run build
        //            ls -la
        //         '''
                
        //     }
        // }
        stage("Test") {
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