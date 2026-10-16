
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

                        mkdir -p ~/.ssh
                        ssh -o BatchMode=yes \
                            "$DEPLOY_USER@$DEPLOY_HOST" \
                            "mkdir -p '$WEB_ROOT'"

                        tar --exclude='.git' -czf website.tar.gz .
                        scp website.tar.gz \
                            "$DEPLOY_USER@$DEPLOY_HOST:/tmp/website.tar.gz"

                        ssh "$DEPLOY_USER@$DEPLOY_HOST" \
                            "tar -xzf /tmp/website.tar.gz -C '$WEB_ROOT' &&
                             rm -f /tmp/website.tar.gz"

                        rm -f website.tar.gz
                    '''
                }
            }
        }
    }
}
