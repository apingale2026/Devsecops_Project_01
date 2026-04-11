pipeline{
    agent{
        label "docker-maven-trivy"
    }
    tools {
        maven 'maven3'
    }
    environment {
        SONAR_IP = '172.31.9.78'
        ECR_REGISTRY = '533267311092.dkr.ecr.ap-south-1.amazonaws.com'
        IMAGE_REPO = "${ECR_REGISTRY}/devsecops1"
        
    }
    stages{
        stage("Trivy FS Scan"){
            steps{
                sh 'trivy fs --exit-code 1 --severity HIGH,CRITICAL .'
            }
        }
        stage ("Build and Scan") {
            steps {
                withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')])
                {
                  sh 'mvn clean verify sonar:sonar \
                -Dsonar.projectKey=devsecops-project \
                -Dsonar.host.url="http://${SONAR_IP}:9000" \
                -Dsonar.token="${SONAR_TOKEN}" \
                -Dsonar.qualitygate.wait=true'
                }
            }
        }
        stage ('ECR Login') {
            steps {
                sh 'aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin $ECR_REGISTRY '
            }
        }
	stage('Build Image') {
      steps {
        sh 'export DOCKER_BUILDKIT=0 && docker build --platform linux/amd64 -t "$IMAGE_REPO:$BUILD_NUMBER" -t "$IMAGE_REPO:latest" .'
      }
    }
    stage('Trivy Image Scan') {
        steps{
        sh 'trivy image --exit-code 1 --severity HIGH,CRITICAL "$IMAGE_REPO:$BUILD_NUMBER"'
        }
    }
    stage('Push Image to ECR') {
        steps {
            sh 'docker push "$IMAGE_REPO:$BUILD_NUMBER"'
            sh  'docker push "$IMAGE_REPO:latest"'
        }
    }
    stage('Update Deployment') {
      steps {
        sh 'sed -i "s|image:.*|image: $IMAGE_REPO:$BUILD_NUMBER|g" deploy-svc.yml'
      }
    }
    stage('Deploy to Kubernetes') 
           {
       steps {
        sh '''#!/bin/bash -l
         aws eks update-kubeconfig \
         --region ap-south-1 \
         --name devsecops-eks \
         --kubeconfig /home/jenkins/.kube/config

         kubectl create ns devsecops
         kubectl apply -f deploy-svc.yml

         kubectl rollout status -n cwvj-devsecops deployment/cwvj-devsecops-demo --timeout=60s || {
         kubectl rollout undo -n cwvj-devsecops deployment/cwvj-devsecops-demo || true
        exit 1
       } 
       '''    
    }
        }
    
    }
    }
