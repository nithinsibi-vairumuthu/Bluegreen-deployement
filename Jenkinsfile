pipeline {
 agent any

 environment {
  AWS_REGION="ap-south-1"
  CLUSTER="devops-eks-cluster"
  IMAGE_TAG="${BUILD_NUMBER}"
 }

 stages {

 stage('Build Image'){
  steps{
   sh "docker build -t prod-app:${IMAGE_TAG} -f app/Dockerfile app/"
  }
 }

 stage('Push ECR'){
  steps{
   sh """
   aws ecr get-login-password --region ${AWS_REGION} \
   | docker login --username AWS --password-stdin 730335674713.dkr.ecr.ap-south-1.amazonaws.com

   docker tag prod-app:${IMAGE_TAG} \
   730335674713.dkr.ecr.ap-south-1.amazonaws.com/prod-app:${IMAGE_TAG}

   docker push 730335674713.dkr.ecr.ap-south-1.amazonaws.com/prod-app:${IMAGE_TAG}
   """
  }
 }

 stage('Deploy GREEN'){
  steps{
   sh """
   aws eks update-kubeconfig --name ${CLUSTER}

   helm upgrade --install eks-demo ./helm/eks-demo \
   --set image.tag=${IMAGE_TAG} \
   --set activeColor=green
   """
  }
 }

  stage('Switch Traffic to BLUE') {
   steps {
    sh """
     kubectl patch service eks-demo-service \
     -p '{"spec":{"selector":{"app":"eks-demo","version":"blue"}}}'
      """
 }
}
 
  stage('Approval'){
  steps{
   input "Switch traffic to GREEN?"
  }
 }

 stage('Switch Traffic'){
  steps{
   sh """
   helm upgrade eks-demo ./helm/eks-demo \
   --set activeColor=green
   """
  }
 }

 }
}
