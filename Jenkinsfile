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
                dir('app') {
                    sh '''
                    docker build -t $ECR_REPO:$IMAGE_TAG .
                    '''
                }
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
                docker push $ECR_REPO:$IMAGE_TAG
                '''
            }
        }
    	stage('Blue-Green Deploy') {
	    steps {
	        sh '''
	        set -e

	        IMAGE=$ECR_REPO:$IMAGE_TAG

	        echo "🔍 Detecting live environment..."

	        if docker ps --format '{{.Names}}' | grep -q flask-blue; then
	            LIVE="blue"
	            NEW="green"
	            NEW_PORT=5002
	        else
	            LIVE="green"
	            NEW="blue"
	            NEW_PORT=5001
	        fi

	        echo "🟢 Live: flask-$LIVE"
	        echo "🔵 New: flask-$NEW on port $NEW_PORT"

	        echo "🧹 Cleaning old NEW container if exists"
	        docker stop flask-$NEW || true
	        docker rm flask-$NEW || true

	        echo "🚀 Starting new container"
	        docker run -d \
	          --name flask-$NEW \
	          -p $NEW_PORT:5000 \
	          --restart always \
	          $IMAGE

	        echo "⏳ Waiting for application to be healthy..."
	        sleep 10

	        echo "🔁 Switching traffic via nginx"
	        sudo sed -i "s/flask-$LIVE/flask-$NEW/" /etc/nginx/conf.d/flask.conf
	        sudo nginx -s reload

	        echo "🛑 Stopping old container flask-$LIVE"
	        docker stop flask-$LIVE || true
	        docker rm flask-$LIVE || true

	        echo "✅ Blue-Green Deployment completed successfully"
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
