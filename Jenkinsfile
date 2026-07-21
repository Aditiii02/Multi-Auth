pipeline {
    agent any

    environment {
        APP_DIR = "/home/ubuntu/Multi-Auth"
        APP_NAME = "multi-auth"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                cd $APP_DIR
                npm install
                '''
            }
        }

        stage('Generate Prisma Client') {
            steps {
                sh '''
                cd $APP_DIR
                npx prisma generate
                '''
            }
        }

        stage('Run Migrations') {
            steps {
                sh '''
                cd $APP_DIR
                npm run migrate
                '''
            }
        }

        stage('Restart Application') {
            steps {
                sh '''
                pm2 restart $APP_NAME
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                sleep 10
                curl --fail http://localhost:5000/
                '''
            }
        }
    }

    post {

        success {
            echo 'Deployment Successful'
        }

        failure {
            echo 'Deployment Failed'
        }
    }
}
