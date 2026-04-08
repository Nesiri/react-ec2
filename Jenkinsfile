pipeline{
    agent any
    environment
    {
        AWS_DOCKER_REGISTRY = '450730497128.dkr.ecr.us-east-2.amazonaws.com/ec2_image'
        APP_NAME = 'ec2_image'
        AWS_DEFAULT_REGION = 'us-east-2'
        CLUSTER_NAME = 'ideal-penguin-3ghbuu'
        SERVICE_NAME = 'react-ec2-task-definition-service-prod'
       TASK_DEF = 'react-ec2-task-definition'
    }

    stages
    {
        
        stage("Build")
        {
            agent
            {
                docker
                {
                    image 'node:20-alpine'
                    reuseNode true
                
                }
           }
            steps
            {
                sh '''
                    npm install
                    npm run build
                    ls -la
                '''
            }
        
        }

        stage("Test")
        {
            agent
            {
                docker
                {
                    image 'node:20-alpine'
                    reuseNode true
                
                }
            }
            steps
            {
                sh '''
                    npm install
                    npm test
                '''
            }
        
        }
        stage('Build My Image')
        {
                agent
                {
                    docker
                    {
                        image 'amazon/aws-cli'
                        reuseNode true
                        args '-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=""'
                    }
                }
                steps
                {
                    withCredentials([usernamePassword(credentialsId: 'S3-ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')])
                    {

                        sh '''
                            dnf install -y docker
                            docker build -t $AWS_DOCKER_REGISTRY/$APP_NAME .
                            docker images

                            aws ecr get-login-password | docker login --username AWS --password-stdin $AWS_DOCKER_REGISTRY
                            docker push $AWS_DOCKER_REGISTRY/$APP_NAME:latest
                        '''
                    }
                }
        }
        stage("Deploy to AWS")
        {
            agent
            {
                docker
                {
                image 'amazon/aws-cli'
                reuseNode true
                args '-u root --entrypoint=""'
                }
            }
            steps{
                withCredentials([usernamePassword(credentialsId: 'S3-ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')])
                 {

                     sh '''
                         echo "Testing AWS CLI"
                aws --version

                # Optional: install jq if needed
                yum install -y jq

                # Update ECS service
                aws ecs update-service \
                    --cluster $CLUSTER_NAME \
                    --service $SERVICE_NAME \
                    --task-definition $TASK_DEF \
                    --force-new-deployment \
                    --region $AWS_DEFAULT_REGION
                     '''

                 }
            }
            
            
        }
    }
    post{
        always{
            echo "========always========"
        }
        success{
            echo "========pipeline executed successfully ========"
        }
        failure{
            echo "========pipeline execution failed========"
        }
    }
}