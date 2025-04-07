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
	    stage ('wait_for_testing'){
	   steps {
		   sh 'pwd; sleep 180; echo "Application Has been deployed on K8S"'
	   	}
	   }
	   
	stage('RunDASTUsingZAP') {
  steps {
    withKubeConfig([credentialsId: 'kubelogin']) {
      script {
        def hostname = sh(script: "kubectl get svc asgbuggy --namespace=devsecops -o json | jq -r '.status.loadBalancer.ingress[0].hostname'", returnStdout: true).trim()
        echo "Running ZAP scan on: http://${hostname}"

        sh """
       docker run -v /var/lib/jenkins/workspace/sonar:/zap/wrk -t owasp/zap2docker-weekly \
       zap.sh -cmd -quickurl http://a8687c7a196454879acf2963caee4333-238419036.us-west-2.elb.amazonaws.com \
       -quickprogress -quickout /zap/wrk/zap_report.html
	"""
      }

      // Archive the report after scanning
      archiveArtifacts artifacts: 'zap_report.html'
    }
  }
}

}
}
