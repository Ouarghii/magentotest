pipeline {
    agent any

    environment {
        MAGENTO_BASE_URL = "http://mage2rock.magento.com"
        DB_HOST = "localhost"
        DB_NAME = "mage2rock"
        DB_USER = "mage2rock"
        DB_PASSWORD = "sarra123"
        ADMIN_USER = "rockadmin"
        ADMIN_PASSWORD = "sarra123"
    }

    stages {
        stage('Checkout') {
            steps {
                // Clone Magento repository
               checkout scm
            }
        }

stage('Install PHP and Composer') {
    steps {
        sh '''
            # Update package list
            apt update
            # Install PHP
            apt install -y php
            
            # Install Composer
            php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
            php composer-setup.php
            mv composer.phar /usr/local/bin/composer
        '''
    }
}

    stage('Install Dependencies') {
        steps {
            sh 'composer install'
    }
}

        stage('Setup Permissions') {
            steps {
                // Set necessary permissions
                sh '''
                    find var generated vendor pub/static pub/media app/etc -type f -exec chmod g+w {} +
                    find var generated vendor pub/static pub/media app/etc -type d -exec chmod g+ws {} +
                '''
            }
        }

        stage('Magento Setup') {
            steps {
                // Run Magento setup install command
                sh '''
                    php bin/magento setup:install \
                        --base-url="${MAGENTO_BASE_URL}" \
                        --db-host="${DB_HOST}" \
                        --db-name="${DB_NAME}" \
                        --db-user="${DB_USER}" \
                        --db-password="${DB_PASSWORD}" \
                        --admin-firstname="Admin" \
                        --admin-lastname="User" \
                        --admin-email="admin@example.com" \
                        --admin-user="${ADMIN_USER}" \
                        --admin-password="${ADMIN_PASSWORD}" \
                        --language="en_US" \
                        --currency="USD" \
                        --timezone="America/Chicago" \
                        --use-rewrites="1"
                '''
            }
        }

        stage('Build Static Content') {
            steps {
                // Generate static content for production
                sh 'php bin/magento setup:static-content:deploy -f'
            }
        }

        stage('Reindex Data') {
            steps {
                // Reindex data
                sh 'php bin/magento indexer:reindex'
            }
        }

        stage('Set Permissions Again') {
            steps {
                // Set permissions for generated and var directories
                sh 'chmod -R 777 var/ pub/ generated/'
            }
        }

        stage('Cache Flush') {
            steps {
                // Clear Magento cache
                sh 'php bin/magento cache:flush'
            }
        }
    }

    post {
        success {
            echo 'Magento build and deployment successful!'
        }
        failure {
            echo 'Magento build or deployment failed.'
        }
    }
}
