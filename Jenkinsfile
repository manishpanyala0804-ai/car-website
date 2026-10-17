
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
                    test -d css
                    test -d images
                    test -d js

                    echo "Website files validated successfully."
                '''
            }
        }

        stage('Package Website') {
            steps {
                sh '''
                    set -eu

                    tar -czf "$WORKSPACE/../website-deploy.tar.gz" \
                        --exclude='.git' \
                        -- *.html css images js

                    echo "All website pages and assets packaged successfully."
                '''
            }
        }

        stage('Deploy to Website Server') {
            steps {
                sshagent(credentials: ['ec2-deploy-key']) {
                    sh '''
                        set -eu

                        echo "Checking connection to Website Server..."

                        ssh -o BatchMode=yes \
                            "$DEPLOY_USER@$DEPLOY_HOST" \
                            "mkdir -p '$WEB_ROOT'"

                        echo "Uploading website files..."

                        scp "$WORKSPACE/../website-deploy.tar.gz" \
                            "$DEPLOY_USER@$DEPLOY_HOST:/tmp/website.tar.gz"

                        echo "Extracting website files..."

                        ssh -o BatchMode=yes \
                            "$DEPLOY_USER@$DEPLOY_HOST" \
                            "tar -xzf /tmp/website.tar.gz -C '$WEB_ROOT' &&
                             rm -f /tmp/website.tar.gz"

                        echo "Website deployed successfully."
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sshagent(credentials: ['ec2-deploy-key']) {
                    sh '''
                        set -eu

                        ssh -o BatchMode=yes \
                            "$DEPLOY_USER@$DEPLOY_HOST" \
                            "test -f '$WEB_ROOT/index.html' &&
                             test -f '$WEB_ROOT/articles.html' &&
                             test -f '$WEB_ROOT/about-us.html' &&
                             test -f '$WEB_ROOT/article.html' &&
                             test -f '$WEB_ROOT/contact-us.html' &&
                             test -f '$WEB_ROOT/sitemap.html'"

                        echo "All required HTML pages verified on the server."
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Car Website deployment completed and verified.'
        }

        failure {
            echo 'FAILED: Check Console Output for the deployment error.'
        }

        always {
            sh 'rm -f "$WORKSPACE/../website-deploy.tar.gz"'
        }
    }
}
