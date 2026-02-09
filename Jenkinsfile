pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"
        AWS_ACCOUNT_ID = "876178095025"
        ECR_REPO_NAME = "flask-app"
        ECR_REPO = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}"
        IMAGE_TAG = "${BUILD_NUMBER}"
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
	        docker build -t flask-app:${BUILD_NUMBER} .
	        docker tag flask-app:${BUILD_NUMBER} \
	          876178095025.dkr.ecr.ap-south-1.amazonaws.com/flask-app:${BUILD_NUMBER}
	        docker tag flask-app:${BUILD_NUMBER} \
	          876178095025.dkr.ecr.ap-south-1.amazonaws.com/flask-app:latest
	        '''
	    }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-ecr-creds'
                ]]) {
                    sh '''
                    aws ecr get-login-password --region $AWS_REGION \
                    | docker login --username AWS --password-stdin $ECR_REPO
                    '''
                }
            }
        }
	
	stage('Push Image to ECR') {
	    steps {
	        withCredentials([[
	            $class: 'AmazonWebServicesCredentialsBinding',
	            credentialsId: 'aws-ecr-creds'
	        ]]) {
	            sh '''
	            aws ecr get-login-password --region ap-south-1 \
	            | docker login --username AWS --password-stdin \
	              876178095025.dkr.ecr.ap-south-1.amazonaws.com

	            docker push 876178095025.dkr.ecr.ap-south-1.amazonaws.com/flask-app:${BUILD_NUMBER}
	            docker push 876178095025.dkr.ecr.ap-south-1.amazonaws.com/flask-app:latest
	            '''
	        }
            }
        }
	stage('Deploy to Kubernetes') {
	  steps {
	    sh '''
	      kubectl set image deployment/flask-app \
	      flask=876178095025.dkr.ecr.ap-south-1.amazonaws.com/flask-app:${BUILD_NUMBER}

	      kubectl rollout status deployment/flask-app
	    '''
	  }
	} 
    }
    post {
        success {
            echo "Pipeline completed successfully 🚀"
        }
        failure {
            echo "Pipeline failed ❌"
        }
    }
}	
