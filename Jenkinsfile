pipeline {
    agent any

    tools { 
        maven 'maven3'  
    }

    environment {
        IMAGE_NAME = 'asg'
        ECR_REGISTRY = '205930608961.dkr.ecr.us-west-2.amazonaws.com'
        ECR_CREDENTIALS = 'ecr:us-west-2:aws-credentials'
    }

    stages {
        stage('Run SCA Analysis Using Snyk') {
            steps {		
                withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
                    sh 'mvn snyk:test -fn'
                }
            }
        }

        stage('Build Docker Image') { 
            steps { 
                script {
                    dockerImage = docker.build("${IMAGE_NAME}")
                }
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    docker.withRegistry("https://${ECR_REGISTRY}", "${ECR_CREDENTIALS}") {
                        dockerImage.push("latest")
                    }
                }
            }
        }
        stage('Kubernetes Deployment of ASG Bugg Web Application') {
	   steps {
	      withKubeConfig([credentialsId: 'kubelogin']) {
		  sh('kubectl delete all --all -n devsecops')
		  sh ('kubectl apply -f deployment.yaml --namespace=devsecops')
		}
	      }
    }
}
}
