pipeline {
  agent any

  tools {
    nodejs 'node-24'
  }

  environment {
    IMAGE_NAME = 'hode0706022/nestjs-jenkins-example'
    IMAGE_TAG = 'latest'
  }

  stages {
    stage('📥 Clone Source Code') {
      steps {
        git 'https://github.com/hode2002/nestjs-jenkins-example.git'
      }
    }

    stage('📦 Install Dependencies') {
      steps {
        sh 'npm install'
      }
    }

    stage('⚙️ Build NestJS App') {
      steps {
        sh 'npm run build'
      }
    }

    stage('🐳 Build & Push Docker Image') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh '''
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
            docker build -t $IMAGE_NAME:$IMAGE_TAG .
            docker push $IMAGE_NAME:$IMAGE_TAG
          '''
        }
      }
    }

    stage('🚀 Deploy to Remote Server') {
      steps {
        withCredentials([string(credentialsId: 'ec2-ip', variable: 'EC2_IP')]) {
            sh '''
                ssh -i $SSH_KEY -o StrictHostKeyChecking=no ec2-user@$EC2_IP '
                cd /home/ec2-user/app &&
                git pull origin main &&
                docker compose pull &&
                docker compose up -d --build
                '
            '''
        }
      }
    }
  }

  post {
    success {
      echo '✅ CI/CD pipeline completed successfully.'
    }
    failure {
      echo '❌ Build or deployment failed.'
    }
  }
}
