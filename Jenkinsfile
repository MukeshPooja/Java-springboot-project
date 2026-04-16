pipeline {
    agent any
    
    tools {
        jdk 'JDK-21'
        maven 'Maven-3.9'
    }
    
    environment {
        JAVA_HOME = "${tool 'JDK-21'}"
        PATH = "${env.JAVA_HOME}/bin:${env.PATH}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
    }
    
    post {
        always {
            // ✅ Fixed: cleanWs() without node {} wrapper
            cleanWs()
        }
        success {
            echo '🎉 Pipeline completed successfully!'
        }
        failure {
            echo '💥 Pipeline failed - check logs'
        }
    }
}