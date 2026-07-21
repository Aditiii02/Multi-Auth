pipeline {
    agent any

    tools {
        nodejs 'NodeJS-20'
    }

    environment {
        APP_DIR = "/home/ubuntu/Multi-Auth"
        HEALTH_URL = "http://localhost:5000/"
    }

    stages {

        stage('Verify Deployment Directory') {
            steps {
                sh '''
                sudo -u ubuntu bash -c "
                    if [ ! -d $APP_DIR/.git ]; then
                        echo 'Application directory not found.'
                        exit 1
                    fi
                "
                '''
            }
        }

        stage('Backup Current Commit') {
            steps {
                sh '''
                sudo -u ubuntu bash -c "
                    cd $APP_DIR
                    git rev-parse HEAD > /tmp/multiauth_previous_commit
                    echo Current commit:
                    cat /tmp/multiauth_previous_commit
                "
                '''
            }
        }

        stage('Pull Latest Code') {
            steps {
                sh '''
                sudo -u ubuntu bash -c "
                    cd $APP_DIR
                    git fetch origin
                    git reset --hard origin/main
                "
                '''
            }
        }

        stage('Copy Environment File') {
            steps {
                withCredentials([file(credentialsId: 'multi-auth-env', variable: 'ENV_FILE')]) {
                    sh '''
                    sudo install \
                        -o ubuntu \
                        -g ubuntu \
                        -m 600 \
                        "$ENV_FILE" \
                        "$APP_DIR/.env"
                    '''
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                sudo -u ubuntu bash -c "
                    cd $APP_DIR
                    npm install
                "
                '''
            }
        }

        stage('Generate Prisma Client') {
            steps {
                sh '''
                sudo -u ubuntu bash -c "
                    cd $APP_DIR
                    npx prisma generate
                "
                '''
            }
        }

        stage('Run Migrations') {
            steps {
                sh '''
                sudo -u ubuntu bash -c "
                    cd $APP_DIR
                    npm run migrate
                "
                '''
            }
        }

        stage('Restart Application') {
            steps {
                sh '''
                sudo -u ubuntu bash -c "
                    cd $APP_DIR
                    pm2 restart multi-auth
                    pm2 save
                "
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                sleep 10
                curl --fail "$HEALTH_URL"
                '''
            }
        }
    }

    post {

        success {
            echo "====================================="
            echo "Deployment Successful"
            echo "====================================="
        }

        failure {

            echo "Deployment Failed."
            echo "Starting Rollback..."

            sh '''
            if [ -f /tmp/multiauth_previous_commit ]; then

                PREVIOUS_COMMIT=$(cat /tmp/multiauth_previous_commit)

                sudo -u ubuntu bash -c "
                    cd $APP_DIR
                    git reset --hard $PREVIOUS_COMMIT
                    npm install
                    pm2 restart multi-auth
                    pm2 save
                "

                echo "Rollback completed."

            else
                echo "Rollback skipped."
            fi
            '''
        }

        always {
            cleanWs()
        }
    }
}
