pipeline {
  agent any

  environment {
    AWS_REGION = "ap-south-1"
    ECR_REPO = "992057087595.dkr.ecr.ap-south-1.amazonaws.com/demo-app"
    IMAGE_TAG = "latest"
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Docker Image') {
      steps {
        sh '''
          docker build -t demo-app:${IMAGE_TAG} .
        '''
      }
    }

    stage('Login to ECR') {
      steps {
        sh '''
          aws ecr get-login-password --region $AWS_REGION | \
          docker login --username AWS \
          --password-stdin $ECR_REPO
        '''
      }
    }

    stage('Tag & Push Image') {
      steps {
        sh '''
          docker tag demo-app:${IMAGE_TAG} $ECR_REPO:${IMAGE_TAG}
          docker push $ECR_REPO:${IMAGE_TAG}
        '''
      }
    }

    stage('Deploy to Primary Cluster') {
      steps {
        sh '''
          kubectl config use-context iam-root-account@eks-prod-primary.ap-south-1.eksctl.io
          kubectl apply -f deployment.yml
        '''
      }
    }

    stage('Deploy to Secondary Cluster') {
      steps {
        sh '''
          kubectl config use-context iam-root-account@eks-prod-secondary.ap-south-1.eksctl.io
          kubectl apply -f deployment.yml
        '''
      }
    }
  }
}

