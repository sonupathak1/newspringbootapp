pipeline {
   tools {
        maven 'Maven3'
    }
    agent any
    environment {
        registry = "544786713028.dkr.ecr.us-east-1.amazonaws.com/sonupathak"
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
      stage('Sonar Analysis') {
      environment {
        scannerHome = tool 'sonar-scanner'
      }
      steps {
        echo '<--------------- Sonar Analysis started  --------------->'
        //         withSonarQubeEnv('sonar-cloud') {
        //         sh "${scannerHome}/bin/sonar-scanner"

        // }
        withSonarQubeEnv('sonar-cloud') {
          sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=sonupathak1 -Dsonar.organization=sonupathak1 -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=eeb64e758a53dbefaf583478e8f641b0155a7b12'
          echo '<--------------- Sonar Analysis stopped  --------------->'
        }
      }
    }
         stage('Quality Gate') {
      steps {
        script {
          echo '<--------------- Quality Gate started  --------------->'
           sleep(10)
          timeout(time: 1, unit: 'MINUTES') {
            def qg = waitForQualityGate()
            if (qg.status != 'OK') {
              error 'Pipeline failed due to the Quality gate issue'
            }
          }
          echo '<--------------- Quality Gate stopped  --------------->'
        }
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
                sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 544786713028.dkr.ecr.us-east-1.amazonaws.com'
                sh 'docker build -t sonupathak .'
                sh 'docker tag sonupathak:latest 544786713028.dkr.ecr.us-east-1.amazonaws.com/sonupathak:latest'
                sh 'docker push 544786713028.dkr.ecr.us-east-1.amazonaws.com/sonupathak:latest'
         }
        }
      }
             stage('Deploy to EKS') {
            steps {
                script {
                    // Authenticate with the EKS cluster (ensure AWS credentials are configured)
                    sh 'aws eks --region us-east-1 update-kubeconfig --name demo-eks'
                    
                    // Apply Kubernetes manifest files to deploy your application
                     // sh "kubectl delete -f eks-deploy-k8s.yaml"
                      sh "kubectl apply -f eks-deploy-k8s.yaml"
                }
            }
        }
    } 
}
