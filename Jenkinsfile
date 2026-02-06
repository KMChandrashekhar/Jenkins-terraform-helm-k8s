pipeline {
  agent any

  environment {
    AWS_REGION = "ap-south-1"
    ECR_REPO   = "<AWS_ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/java-app"
    CLUSTER    = "demo-eks"
  }

  stages {

    stage('Checkout') {
      steps {
        git branch: 'main',
            url: 'https://github.com/<YOUR_GITHUB_USERNAME>/eks-jenkins-helm-project.git'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker build -t $ECR_REPO:latest .'
      }
    }

    stage('Login to ECR') {
      steps {
        sh '''
        aws ecr get-login-password --region $AWS_REGION |
        docker login --username AWS --password-stdin $ECR_REPO
        '''
      }
    }

    stage('Push Image to ECR') {
      steps {
        sh 'docker push $ECR_REPO:latest'
      }
    }

    stage('Update kubeconfig') {
      steps {
        sh '''
        aws eks update-kubeconfig \
          --region $AWS_REGION \
          --name $CLUSTER
        '''
      }
    }

    stage('Deploy using Helm') {
      steps {
        sh '''
        helm upgrade --install java-app helm-chart \
          --set image.repository=$ECR_REPO \
          --set image.tag=latest
        '''
      }
    }
  }
}
