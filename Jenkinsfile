pipeline {
    agent any
    environment {
        AWS_ACCOUNT_ID="176295807911"
        AWS_DEFAULT_REGION="us-east-1" 
	CLUSTER_NAME="Nodejs"
	SERVICE_NAME="nodejsapp-service"
	TASK_DEFINITION_NAME="nodejs-app"
	DESIRED_COUNT="1"
        IMAGE_REPO_NAME="nodejs"
        IMAGE_TAG="${env.BUILD_ID}"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
	registryCredential = "176295807911"
    }
   
    stages {

    // Tests
    stage('Unit Tests') {
      steps{
        script {
          sh 'npm install'
	  sh 'npm test -- --watchAll=false'
        }
      }
    }
        
    stage("Build Docker image") {
        steps {
            sh "docker build -t ${env.IMAGE_REPO_NAME}:${env.IMAGE_TAG} ."
        }
    }
   
    // Uploading Docker images into AWS ECR
    stage("Push Docker image to ECR") {
        steps {
            withCredentials([
                [
                    $class: 'AmazonWebServicesCredentialsBinding',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY',
                    credentialsId: '176295807911'
                ]
            ]) {
                sh "aws ecr get-login-password --region ${env.AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_DEFAULT_REGION}.amazonaws.com"
		sh "docker tag ${env.IMAGE_REPO_NAME}:${env.IMAGE_TAG} ${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_DEFAULT_REGION}.amazonaws.com/${env.IMAGE_REPO_NAME}:${env.IMAGE_TAG}"
                sh "docker push ${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_DEFAULT_REGION}.amazonaws.com/${env.IMAGE_REPO_NAME}:${env.IMAGE_TAG}"
            }
        }
    }
    stage("Update ECS service") {
        steps {
               withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',
                        accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY',
                        credentialsId: '176295807911'
                    ]]) {
                    sh "aws ecs update-service --region us-east-1 --cluster Nodejs --service nodeapp-service"
                    sh "aws ecs update-service --region us-east-1 --cluster ${env.CLUSTER_NAME} --service ${env.SERVICE_NAME}"
                }
            }
        }     
      
    }
}

