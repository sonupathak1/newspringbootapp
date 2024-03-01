pipeline {
   tools {
        maven 'Maven3'
    }
    agent any
    environment {
        registry = "779870982142.dkr.ecr.us-west-1.amazonaws.com/sonupathak"
    }
   
    stages {
        stage('Cloning Git') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '', url: 'https://github.com/sonupathak1/newspringbootapp.git']]])     
            }
        }
      stage ('Build') {
          steps {
            sh 'mvn clean install'           
            }
      }
      stage('Unit Test') {
            steps {
                echo '<--------------- Unit Testing started  --------------->'
                sh 'mvn surefire-report:report'
                echo '<------------- Unit Testing stopped  --------------->'
      }
    }

      stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    sh 'docker build -t myrepo .'
                }
            }
    }
    // Uploading Docker images into AWS ECR
    stage('Pushing to ECR') {
     steps{  
         script {
                sh 'aws ecr get-login-password --region us-west-1 | docker login -u AWS -p $(aws ecr get-login-password --region us-west1) 779870982142.dkr.ecr.us-west-1.amazonaws.com'
                sh 'docker build -t sonupathak .'
                sh 'docker tag sonupathak:latest 779870982142.dkr.ecr.us-west-1.amazonaws.com/sonupathak:latest'
                sh 'docker push 779870982142.dkr.ecr.us-west-1.amazonaws.com/sonupathak:latest'
         }
        }
      }
             stage('Deploy to EKS') {
            steps {
                script {
                    // Authenticate with the EKS cluster (ensure AWS credentials are configured)
                    sh 'aws eks --region us-west-1 update-kubeconfig --name demo-eks'
                    
                    // Apply Kubernetes manifest files to deploy your application
                     // sh "kubectl delete -f eks-deploy-k8s.yaml"
                      sh "kubectl apply -f eks-deploy-k8s.yaml"
                }
            }
        }
    } 
}
