pipeline {
    agent any
    
    tools {nodejs "NodeJS"}
    
    environment {
    AWS_ACCOUNT_ID="399348531980"
    AWS_DEFAULT_REGION="us-east-1"
    registryCredential = "sree"
    }
   
  stages { 

    stage('Unit Test') {
        steps {
            sh 'npm install --package-lock-only'
            sh 'npm test'
        }
    }


    // Build and Push images to ECR
    stage('Build and Push images to ECR') {
      steps{
        script {
	  sh './start.sh build'
	  sh 'docker images'
          sh './start.sh push'
        }
      }
    }
      
    // Deploy infra provision yaml files
    stage('Submit Stack') {
      steps{
        script {
	      sh 'aws --version'
          sh 'aws cloudformation deploy --template-file infrastructure/cloudformation/01_vpc-network.yml --region us-east-1 --stack-name msdemo-vpc --capabilities CAPABILITY_NAMED_IAM'
          sh 'aws cloudformation deploy --template-file infrastructure/cloudformation/02_nat-gateway.yml --region us-east-1 --stack-name msdemo-nat --capabilities CAPABILITY_NAMED_IAM'
          sh 'aws cloudformation deploy --template-file infrastructure/cloudformation/03_load-balancer.yml --region us-east-1 --stack-name msdemo-lb --capabilities CAPABILITY_NAMED_IAM'
          sh 'aws cloudformation deploy --template-file infrastructure/cloudformation/04_ecs-cluster.yml --region us-east-1 --stack-name msdemo-ecscluster --capabilities CAPABILITY_NAMED_IAM'
        }
      }
    }
   
    // Deploy microservices in Fargate
    stage('Deploy to ECS Fargate') {
     steps{  
         script {
           sh 'aws cloudformation deploy --template-file infrastructure/cloudformation/05_customer-service.1.yml --region us-east-1 --stack-name msdemo-customerservice --capabilities CAPABILITY_NAMED_IAM'
           sh 'aws cloudformation deploy --template-file infrastructure/cloudformation/06_order-service.yml --region us-east-1 --stack-name msdemo-orderservice --capabilities CAPABILITY_NAMED_IAM'
         }
        }
      }    
    }
}
