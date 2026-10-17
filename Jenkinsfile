
pipeline {
    agent any

    environment {
        DEPLOY_HOST = '172.31.43.117'
        DEPLOY_USER = 'deploy'
        WEB_ROOT = '/var/www/html'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate Website') {
            steps {
                sh '''
                    set -eu
                    test -f index.html
                    echo "Website files validated successfully."
                '''
            }
        }

        stage('Deploy to Website Server') {
            steps {
                sshagent(credentials: ['ec2-deploy-key']) {
                    sh '''
                        set -eu

                        ssh -o BatchMode=yes \
                            "$DEPLOY_USER@$DEPLOY_HOST" \
                            "mkdir -p '$WEB_ROOT'"

                        tar -czf "$WORKSPACE/../website-deploy.tar.gz" \
                            --exclude='.git' \
                            --exclude='website.tar.gz' \
                            index.html css images js

                        scp "$WORKSPACE/../website-deploy.tar.gz" \
                            "$DEPLOY_USER@$DEPLOY_HOST:/tmp/website.tar.gz"

                        ssh -o BatchMode=yes \
                            "$DEPLOY_USER@$DEPLOY_HOST" \
                            "tar -xzf /tmp/website.tar.gz -C '$WEB_ROOT' &&
                             rm -f /tmp/website.tar.gz"

                        rm -f "$WORKSPACE/../website-deploy.tar.gz"

                        echo "Website deployed successfully."
                    '''
                }
            }
        }
    }
}
