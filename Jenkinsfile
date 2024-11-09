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
        PHP_BIN_PATH = "/path/to/php"  // Update this to your PHP binary location
    }

    stages {
        stage('Checkout') {
            steps {
                // Clone Magento repository
                checkout scm
            }
        }
        stage('Install Dependencies for PHP Build') {
            steps {
                // Manually install dependencies without sudo
                sh '''
                    # Install GCC and Make (if not already installed)
                    if ! command -v gcc &> /dev/null; then
                        echo "GCC not found, downloading and installing..."
                        curl -LO http://ftp.gnu.org/gnu/gcc/gcc-9.3.0/gcc-9.3.0.tar.gz
                        tar -xzf gcc-9.3.0.tar.gz
                        cd gcc-9.3.0
                        ./contrib/download_prerequisites
                        mkdir build
                        cd build
                        ../configure --prefix=$HOME/gcc
                        make -j"$(nproc)"
                        make install
                        export PATH=$HOME/gcc/bin:$PATH
                        cd ..
                    fi
                    
                    # Install Make (if not already installed)
                    if ! command -v make &> /dev/null; then
                        echo "Make not found, downloading and installing..."
                        curl -LO https://ftp.gnu.org/gnu/make/make-4.3.tar.gz
                        tar -xzf make-4.3.tar.gz
                        cd make-4.3
                        ./configure --prefix=$HOME/make
                        make -j"$(nproc)"
                        make install
                        export PATH=$HOME/make/bin:$PATH
                        cd ..
                    fi
                    
                    # Install necessary libraries like OpenSSL, Curl, and XML
                    # Download and compile OpenSSL
                    if ! command -v openssl &> /dev/null; then
                        echo "OpenSSL not found, downloading and installing..."
                        curl -LO https://www.openssl.org/source/openssl-1.1.1l.tar.gz
                        tar -xzf openssl-1.1.1l.tar.gz
                        cd openssl-1.1.1l
                        ./config --prefix=$HOME/openssl
                        make -j"$(nproc)"
                        make install
                        export PATH=$HOME/openssl/bin:$PATH
                        cd ..
                    fi
                    
                    # Download and compile Curl
                    if ! command -v curl &> /dev/null; then
                        echo "Curl not found, downloading and installing..."
                        curl -LO https://curl.haxx.se/download/curl-7.79.1.tar.gz
                        tar -xzf curl-7.79.1.tar.gz
                        cd curl-7.79.1
                        ./configure --prefix=$HOME/curl
                        make -j"$(nproc)"
                        make install
                        export PATH=$HOME/curl/bin:$PATH
                        cd ..
                    fi
                    
                    # Download and compile libxml2
                    if ! command -v xmllint &> /dev/null; then
                        echo "libxml2 not found, downloading and installing..."
                        curl -LO http://xmlsoft.org/sources/libxml2-2.9.12.tar.gz
                        tar -xzf libxml2-2.9.12.tar.gz
                        cd libxml2-2.9.12
                        ./configure --prefix=$HOME/libxml2
                        make -j"$(nproc)"
                        make install
                        export PATH=$HOME/libxml2/bin:$PATH
                        cd ..
                    fi
                '''
            }
        }
        stage('Install PHP and Composer') {
            steps {
                script {
                    // Download PHP Binary if not already available
                    sh '''
                        if [ ! -f ${PHP_BIN_PATH}/php ]; then
                            echo "PHP binary not found, downloading..."
                            curl -LO https://www.php.net/distributions/php-8.1.0.tar.gz
                            tar -xzf php-8.1.0.tar.gz
                            cd php-8.1.0
                            ./configure --prefix=${PHP_BIN_PATH} --enable-fpm --with-openssl --with-curl --enable-mbstring --with-mysqli
                            make
                            make install
                        fi
                    '''
                }
                // Install Composer globally
                sh '''
                    curl -sS https://getcomposer.org/installer | php
                    mv composer.phar /usr/local/bin/composer
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                // Install Magento dependencies via Composer
                sh 'composer install'
            }
        }

        stage('Setup Permissions') {
            steps {
                sh '''
                    find var generated vendor pub/static pub/media app/etc -type f -exec chmod g+w {} +
                    find var generated vendor pub/static pub/media app/etc -type d -exec chmod g+ws {} +
                '''
            }
        }

        stage('Magento Setup') {
            steps {
                sh '''
                    ${PHP_BIN_PATH}/bin/magento setup:install \
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
                sh '${PHP_BIN_PATH}/bin/magento setup:static-content:deploy -f'
            }
        }

        stage('Reindex Data') {
            steps {
                sh '${PHP_BIN_PATH}/bin/magento indexer:reindex'
            }
        }

        stage('Set Permissions Again') {
            steps {
                sh 'chmod -R 777 var/ pub/ generated/'
            }
        }

        stage('Cache Flush') {
            steps {
                sh '${PHP_BIN_PATH}/bin/magento cache:flush'
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
