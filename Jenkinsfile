pipeline {
    agent{
        docker{
            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
            reuseNode true
        }
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
                    netlify --version
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
