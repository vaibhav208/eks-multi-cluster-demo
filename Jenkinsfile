pipeline {
  agent any

  environment {
    AWS_REGION = "ap-south-1"
    ECR_REPO   = "992057087595.dkr.ecr.ap-south-1.amazonaws.com/demo-app"
    IMAGE_TAG  = "latest"
  }

  stages {

    stage('Checkout Code') {
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

    stage('Login to Amazon ECR') {
      steps {
        sh '''
          aws ecr get-login-password --region ${AWS_REGION} | \
          docker login --username AWS \
          --password-stdin ${ECR_REPO}
        '''
      }
    }

    stage('Tag & Push Image to ECR') {
      steps {
        sh '''
          docker tag demo-app:${IMAGE_TAG} ${ECR_REPO}:${IMAGE_TAG}
          docker push ${ECR_REPO}:${IMAGE_TAG}
        '''
      }
    }

    stage('Deploy to Primary EKS Cluster') {
      steps {
        sh '''
          aws eks update-kubeconfig \
            --region ${AWS_REGION} \
            --name eks-prod-primary

          kubectl apply -f deployment.yml
        '''
      }
    }

    stage('Deploy to Secondary EKS Cluster') {
      steps {
        sh '''
          aws eks update-kubeconfig \
            --region ${AWS_REGION} \
            --name eks-prod-secondary

          kubectl apply -f deployment.yml
        '''
      }
    }
  }

  post {
    success {
      echo "✅ CI/CD pipeline completed successfully. App deployed to both clusters."
    }
    failure {
      echo "❌ CI/CD pipeline failed. Check logs above."
    }
  }
}

