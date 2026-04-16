pipeline {
    agent any

    tools {
        jdk 'JDK-21'
        maven 'Maven-3.9'
    }

    environment {
        // Set JAVA_HOME for Maven
        JAVA_HOME = "${tool 'JDK-17'}"
        PATH = "${env.JAVA_HOME}/bin:${env.PATH}"
        
        // App configuration
        APP_NAME = 'springboot-app'
        APP_PORT = '8081'
        ARTIFACT_BUCKET = 'your-s3-artifacts-bucket' // Optional
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo "✅ Checked out: ${env.GIT_COMMIT}"
            }
        }

        stage('Validate') {
            steps {
                echo '🔍 Validating project structure...'
                sh 'ls -la'
                sh 'cat pom.xml | head -20'
            }
        }

        stage('Build') {
            steps {
                echo '🔨 Building with Maven...'
                sh 'mvn clean package -DskipTests'
            }
            post {
                success {
                    echo '✅ Build successful'
                    archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: false
                }
                failure {
                    error '❌ Build failed - check Maven logs'
                }
            }
        }

        stage('Test') {
            when {
                expression { params.RUN_TESTS == true }
            }
            steps {
                echo '🧪 Running tests...'
                sh 'mvn test'
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package for Deployment') {
            steps {
                echo '📦 Preparing deployment artifact...'
                sh '''
                    mkdir -p deploy
                    cp target/*.jar deploy/${APP_NAME}.jar
                    cat > deploy/run.sh << 'EOF'
#!/bin/bash
java -jar ${APP_NAME}.jar --server.port=${APP_PORT}
EOF
                    chmod +x deploy/run.sh
                '''
            }
        }

        stage('Deploy to EC2') {
            when {
                branch 'main'
            }
            steps {
                echo '🚀 Deploying to EC2...'
                withAWS(credentials: 'aws-prod-credentials', region: 'us-east-1') {
                    // Option 1: SCP + SSH deployment
                    sh '''
                        # Copy artifact to EC2 (replace with your target IP)
                        # scp -i key.pem target/*.jar ec2-user@target-ec2:/opt/app/
                        
                        # Or use AWS Systems Manager for agentless deployment
                        aws ssm send-command \
                          --instance-ids "i-xxxxxxxx" \
                          --document-name "AWS-RunShellScript" \
                          --parameters 'commands=["cd /opt/app && java -jar *.jar --server.port=8081 &"]'
                    '''
                }
            }
        }

        stage('Health Check') {
            when {
                branch 'main'
            }
            steps {
                echo '🏥 Verifying deployment...'
                sh '''
                    # Wait for app to start
                    sleep 30
                    # Health check
                    curl -f http://localhost:${APP_PORT}/actuator/health || exit 1
                '''
            }
        }
    }

    post {
        always {
            echo '🧹 Cleaning workspace...'
            cleanWs()
        }
        success {
            echo '🎉 Pipeline completed successfully!'
            // Optional: Slack/email notification
        }
        failure {
            echo '💥 Pipeline failed!'
            // Optional: Alert on failure
        }
    }
}