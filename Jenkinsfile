pipeline {
    agent{
        docker{
            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
            reuseNode true
        }
    }
    environment {
        NETLIFY_SITE_ID = '8fe2b9d3-04ba-457d-bbe3-5932b368db4a'
    }
    stages {
        stage('Build1') {

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
        stage('Test'){
            steps{
                echo 'Test stage'  
                sh '''
                    test -f build/index.html
                    npm test 
                '''

            }
        }
        stage('E2E') {
            steps {
                sh '''
                    npm install serve
                    # 서버 실행 및 PID 저장
                    npx serve -s build > /dev/null 2>&1 &
                    SERVER_PID=$!
                    
                    # 테스트 실행
                    npx playwright test --reporter=html
                    
                    # 테스트 완료 후 서버 종료
                    kill $SERVER_PID
                '''
            }
        }
        stage('Deploy') {
            steps {
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                    echo "프로젝트 배포중.. 사이트아이디 : $NETLIFY_SITE_ID"
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
