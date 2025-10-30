pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    stages {

        stage('Checkout') {
            steps {
                echo '📦 Cloning repository...'
                git branch: 'main', url: 'https://github.com/harisai2005/maven-demo.git'
            }
        }

        stage('Build') {
            steps {
                echo '🔨 Building the application...'
                sh 'mvn clean package'
            }
        }

        stage('Test') {
            steps {
                echo '🧪 Running tests...'
                sh 'mvn test'
            }
        }

        stage('Deploy') {
            steps {
                echo '🚀 Starting deployment process...'

                // Create deployment directory (if not exists)
                sh 'mkdir -p /var/lib/jenkins/deploy'

                // Copy the generated jar from target directory
                sh 'cp target/*.jar /var/lib/jenkins/deploy/'

                // Stop previously running instance (if any)
                sh '''
                    PID_FILE="/var/lib/jenkins/deploy/app.pid"
                    if [ -f "$PID_FILE" ]; then
                        OLD_PID=$(cat "$PID_FILE")
                        if ps -p $OLD_PID > /dev/null; then
                            echo "Stopping old application process: $OLD_PID"
                            kill $OLD_PID
                        fi
                        rm -f "$PID_FILE"
                    fi
                '''

                // Start new application in background & record PID
                sh '''
                    nohup java -jar /var/lib/jenkins/deploy/maven-demo-1.0-SNAPSHOT.jar \
                    > /var/lib/jenkins/deploy/app.log 2>&1 &
                    echo $! > /var/lib/jenkins/deploy/app.pid
                    echo "New application started with PID: $(cat /var/lib/jenkins/deploy/app.pid)"
                '''

                echo '✅ Application deployed successfully and running in background.'
            }
        }
    }

    post {
        success {
            echo '✅ Build, Test, and Deployment completed successfully!'
            echo 'You can verify the deployment log using:'
            echo '   cat /var/lib/jenkins/deploy/app.log'
        }
        failure {
            echo '❌ Build failed. Deployment skipped.'
        }
    }
}

