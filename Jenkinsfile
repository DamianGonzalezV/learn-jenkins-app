pipeline {
  agent any

  stages {
    // stage('Build') {
    //   agent {
    //     docker {
    //       image 'node:18-alpine'
    //       reuseNode true
    //     }
    //   }
    //   steps {
    //     sh '''
    //       ls -la
    //       node --version
    //       npm --version
    //       npm ci
    //       npm run build
    //       ls -la
    //     '''
    //   }
    // }
    stage('Run tests'){
      parallel {
        // everything in this block will run in parallel 
        stage('Unit tests'){
          agent {
            docker {
              image 'node:18-alpine'
              reuseNode true
            }
          }
          steps {
            sh '''
              echo "Test stage"
              npm test
            '''
          }
        }
        stage('E2E Tests'){
          agent {
            docker {
              image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
              reuseNode true
              // args '-u root:root'
            }
          }
          steps {
            sh '''
              npm install serve
              node_modules/.bin/serve -s build &
              sleep 8
              npx playwright test --reporter=html
            '''
          }
        }
      }
    }

    post {
      always {
      junit 'jest-results/junit.xml'
      junit 'test-results-e2e/playwright-junit-results.xml'
      publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
      }
    }
  }
}