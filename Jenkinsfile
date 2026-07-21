pipeline {
    agent any

    environment {
        APP_NAME = "multi-auth"
    }

    stages {

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Generate Prisma Client') {
            steps {
                sh 'npx prisma generate'
            }
        }

        stage('Run Migrations') {
            steps {
                sh 'npm run migrate'
            }
        }

        stage('Restart Application') {
            steps {
                sh 'pm2 restart multi-auth'
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
