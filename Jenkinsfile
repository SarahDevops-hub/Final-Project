pipeline {
    agent any
    stages {
        stage('Docker Compose Up') {
            steps {
                sh '''
                # 1. Force remove specific containers if they exist
                docker rm -f mysql_db wordpress_app wp_cli 2>/dev/null || true
                
                # 2. Full compose down with cleanup
                # docker-compose down --volumes --remove-orphans --timeout 1 2>/dev/null || true
                
                # 3. Wait to ensure cleanup completes
                sleep 2

                docker-compose up -d
                '''
            }
        }
        stage('Wait for WordPress') {
            steps {
                script {
                    def maxAttempts = 20
                    def attempt = 1
                    while (attempt <= maxAttempts) {
                        try {
                            sh 'curl http://localhost:8080'
                            echo "WordPress is ready!"
                            break
                        } catch (Exception e) {
                            echo "Attempt ${attempt}: WordPress not ready yet, waiting..."
                            sleep time: 10, unit: 'SECONDS'
                            attempt++
                        }
                    }
                    if (attempt > maxAttempts) {
                        error "WordPress did not become ready after ${maxAttempts} attempts."
                    }
                }
            }
        }
        stage('Verify WordPress Page') {
          steps {
            sh 'curl http://localhost:8080'
          }
        }
        stage('Wait for WP-CLI Initialization') {
            steps {
                sleep time: 10, unit: 'SECONDS' // Adjust the wait time as needed
                echo "WP-CLI should have initialized WordPress."
            }
        }
        stage('Run WP-CLI Tests') {
            steps {
                sh '''
                docker-compose exec -T --user=33:33 wp-cli bash -c '
                wp --require=/var/www/html/wp-cli-test-command.php test


                '
                '''

            }
        }
        stage('Remove WP-CLI Container') {
            steps {
                sh 'docker rm -f wp_cli || true'
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}


