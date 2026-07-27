pipeline {
    // 전역 에이전트를 사용하지 않음으로써 컨테이너 중첩 방지
    agent none 

    environment{
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        APP_NAME = 'myjenkinsapp'
        AWS_DEFAULT_REGION = 'ap-northeast-2'
        AWS_DOCKER_REGISTRY='476141957383.dkr.ecr.ap-northeast-2.amazonaws.com/myjenkinsapp'
        AWS_ECS_CLUSTER = 'decent-bee-255jix'
        AWS_ECS_SERVICE_PROD = 'LearnJenkinsApp-Service-Prod'
        AWS_ECS_TD_PROD = 'LearnJenkinsApp-TaskDefinition-Prod'
    }

    stages {
        
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
        
        // 빌드 이후에 수행
        stage('Build Docker image') {
            agent {
                docker { 
                    image 'amazon/aws-cli'
                    reuseNode true
                    // ★ 수정 1: Docker 소켓 마운트 및 root 권한 부여
                    args "-u root --entrypoint='' -v /var/run/docker.sock:/var/run/docker.sock"
                }
            }
            steps {
                // ★ 수정 2: ECR 로그인 및 Push를 위해 이 스테이지에도 크리덴셜 바인딩 추가
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        export AWS_DEFAULT_REGION=$AWS_DEFAULT_REGION
                        
                        yum install -y docker git
                        
                        # 애플 실리콘칩(M1,M2 등) 사용자는 해당 옵션 붙이기
                        docker build --platform linux/amd64 -t $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION .
                        
                        aws ecr get-login-password | docker login --username AWS --password-stdin $AWS_DOCKER_REGISTRY
                        docker push $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION
                    '''
                }
            }
        }

        stage('Deploy to AWS') {
            agent {
                docker { 
                    image 'amazon/aws-cli'
                    reuseNode true
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

                        aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE_PROD --task-definition $AWS_ECS_TD_PROD:$LATEST_TD_REVISION

                        aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE_PROD
                    '''
                }
            }
        }
    }
}