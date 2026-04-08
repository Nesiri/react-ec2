pipeline{
    agent any
      environment {
        AWS_ACCOUNT_ID = '450730497128'
        AWS_DOCKER_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.us-east-2.amazonaws.com"
        APP_NAME = 'react-ec2'  // Must match your ECR repository
        AWS_DEFAULT_REGION = 'us-east-2'
        CLUSTER_NAME = 'ideal-penguin-3ghbuu'
        SERVICE_NAME = 'react-ec2-task-definition-service-prod'
        TASK_DEF_FAMILY = 'react-ec2-task-definition'
        FULL_IMAGE_NAME = "${AWS_DOCKER_REGISTRY}/${APP_NAME}:latest"
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
                        
                        # Build image
                        docker build -t ${APP_NAME}:latest .
                        docker tag ${APP_NAME}:latest ${FULL_IMAGE_NAME}
                        
                        echo "Built image: ${FULL_IMAGE_NAME}"
                        docker images
                        
                        # Login to ECR
                        aws ecr get-login-password | docker login --username AWS --password-stdin ${AWS_DOCKER_REGISTRY}
                        
                        # Push to ECR
                        docker push ${FULL_IMAGE_NAME}
                        
                        echo "Successfully pushed ${FULL_IMAGE_NAME}"
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
                         aws --version
                        yum install jq -y

                        # Update the task definition with the correct image name
                        # Change ec2_image to react-ec2 in your task-definition.json file
                        LATEST_TD_REVISION=$(aws ecs register-task-definition \
                            --family ${TASK_DEF_FAMILY} \
                            --cli-input-json file://task-definition.json \
                            --query 'taskDefinition.revision' \
                            --output text)  
                        
                        aws ecs update-service \
                            --cluster ideal-penguin-3ghbuu \
                            --service react-ec2-task-definition-service-prod \
                            --task-definition react-ec2-task-definition:$LATEST_TD_REVISION \
                            --force-new-deployment
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