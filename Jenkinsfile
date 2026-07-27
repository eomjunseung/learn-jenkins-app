pipeline {
    // 전역 에이전트를 사용하지 않음으로써 컨테이너 중첩 방지
    agent none 

    environment{
        AWS_DEFAULT_REGION = 'ap-northeast-2'
    }

    stages {
        stage('Deploy to AWS') {
            agent {
                docker { 
                    image 'amazon/aws-cli'
                    // aws-cli 이미지는 기본적으로 실행 후 바로 종료되므로 엔트리포인트 무력화
                    reuseNode true
                
                    // jq 설치를 위하여 root 계정으로 변경
                    args "-u root --entrypoint=''" 
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh'''
                        export AWS_DEFAULT_REGION=$AWS_DEFAULT_REGION

                        aws --version

                        yum install jq -y

                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')

                        echo $LATEST_TD_REVISION

                        aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision'
                        aws ecs update-service --cluster decent-bee-255jix --service LearnJenkinsApp-Service-Prod --task-definition LearnJenkinsApp-TaskDefinition-Prod:$LATEST_TD_REVISION
                    '''
                }
            }
        }

        stage('Build') {
            agent {
                docker { 
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy' 
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo '빌드 시작..'
                    node --version
                    npm --version
                    npm ci
                    npm run build
                '''
            }
        }

        

    }
}
