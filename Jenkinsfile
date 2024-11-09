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
                // Checkout the source code from the repository
                checkout scm
            }
        }

          stage('Install PHP and Composer') {
            steps {
                sh '''
                    # Download PHP source tarball using curl
                    curl -LO https://www.php.net/distributions/php-8.1.0.tar.gz
                    
                    # Extract the tarball using tar with gzip compression
                    tar -xzf php-8.1.0.tar.gz
                    
                    # Navigate into the PHP source directory
                    cd php-8.1.0
                    
                    # Configure and install PHP in a local directory
                    ./configure --prefix=$HOME/php --enable-fpm --with-openssl --with-curl --enable-mbstring --with-mysqli
                    make -j"$(nproc)"
                    make install
                    
                    # Set PHP binaries path for current session
                    export PATH=$HOME/php/bin:$PATH
                    
                    # Verify PHP installation
                    php -v
                    
                    # Install Composer
                    curl -sS https://getcomposer.org/installer | php -- --install-dir=$HOME --filename=composer
                    
                    # Add Composer to PATH for the current session
                    export PATH=$HOME:$PATH
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                // Install PHP dependencies using Composer
                sh 'composer install'
            }
        }

        stage('Setup Permissions') {
            steps {
                // Set necessary file and directory permissions for Magento
                sh '''
                    find var generated vendor pub/static pub/media app/etc -type f -exec chmod g+w {} +
                    find var generated vendor pub/static pub/media app/etc -type d -exec chmod g+ws {} +
                '''
            }
        }

        stage('Magento Setup') {
            steps {
                // Run Magento setup and install command
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
                // Deploy static content for Magento
                sh 'php bin/magento setup:static-content:deploy -f'
            }
        }

        stage('Reindex Data') {
            steps {
                // Reindex Magento data
                sh 'php bin/magento indexer:reindex'
            }
        }

        stage('Set Permissions Again') {
            steps {
                // Set file and directory permissions again after Magento setup
                sh 'chmod -R 777 var/ pub/ generated/'
            }
        }

        stage('Cache Flush') {
            steps {
                // Flush Magento cache
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
