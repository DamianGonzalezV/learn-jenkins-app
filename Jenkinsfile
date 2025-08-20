pipeline {
  agent any

  environment {
    NETLIFY_SITE_ID = '073a9045-c4f7-43a6-be3d-2650ff195465'
    NETLIFY_AUTH_TOKEN = credentials('netlify-token')
  }

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
          ls -la
          node --version
          npm --version
          npm ci
          npm run build
          ls -la
        '''
      }
    }
    
    stage('Tests'){
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
          post {
            always {
              junit 'test-results-e2e/playwright-junit-results.xml'
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
          npm install netlify-cli
          node_modules/.bin/netlify --version
          echo "Deploying to production. SITE ID: $NETLIFY_SITE_ID"
          node_modules/.bin/netlify status
          node_modules/.bin/netlify deploy
          node_modules/.bin/netlify --dir=build --prod
        '''
      }
    }
  }
}