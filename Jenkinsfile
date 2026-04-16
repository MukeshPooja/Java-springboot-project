pipeline {
    agent any
    
    tools {
        // ✅ Use correct tool names from Global Tool Configuration
        jdk 'JDK-21'              // ← Verify this name exists in Jenkins
        maven 'Maven-3.9'
    }
    
    environment {
        JAVA_HOME = "${tool 'JDK-21'}"
        PATH = "${env.JAVA_HOME}/bin:${env.PATH}"
    }
    
    stages {
        stage('Build') {
            steps {
                sh 'java -version'
                sh 'mvn -version'
                sh 'mvn clean package -DskipTests'
            }
        }
    }
    
    post {
        always {
            // ✅ Fix: cleanWs must run inside node context
            node {
                cleanWs()
            }
        }
        success {
            echo '🎉 Build successful!'
        }
        failure {
            echo '💥 Build failed - check logs'
        }
    }
}