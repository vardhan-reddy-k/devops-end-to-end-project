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
	        sh '''
	        aws ecr get-login-password --region ap-south-1 \
	        | docker login --username AWS --password-stdin \
	          876178095025.dkr.ecr.ap-south-1.amazonaws.com

	        docker push 876178095025.dkr.ecr.ap-south-1.amazonaws.com/flask-app:${BUILD_NUMBER}
	        docker push 876178095025.dkr.ecr.ap-south-1.amazonaws.com/flask-app:latest
	        '''
            }
        }

	stage('Deploy Blue-Green') {
	    steps {
	        sh '''
	        set -e

	        IMAGE=$ECR_REPO:$IMAGE_TAG

	        if docker ps --format '{{.Names}}' | grep -q flask-blue; then
	            LIVE="blue"
	            NEW="green"
	            LIVE_PORT=5001
	            NEW_PORT=5002
	        else
	            LIVE="green"
	            NEW="blue"
	            LIVE_PORT=5002
	            NEW_PORT=5001
	        fi

	        echo "Live: flask-$LIVE on port $LIVE_PORT"
	        echo "Deploying: flask-$NEW on port $NEW_PORT"

	        docker stop flask-$NEW || true
	        docker rm flask-$NEW || true

	        docker run -d \
	          --name flask-$NEW \
	          -p $NEW_PORT:5000 \
	          --restart always \
	          $IMAGE

	        sleep 10

	        sudo nginx -t
	        sudo sed -i "s/$LIVE_PORT/$NEW_PORT/" /etc/nginx/conf.d/flask.conf
	        sudo systemctl reload nginx

	        docker stop flask-$LIVE || true
	        docker rm flask-$LIVE || true

	        echo "Blue-Green deployment successful"
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
