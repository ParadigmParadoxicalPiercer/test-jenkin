// Jenkinsfile
pipeline {
    agent any
    stages {
        stage('Clone') {
            steps {
                git branch: 'main', url: 'https://github.com/ParadigmParadoxicalPiercer/test-jenkin.git'
            }
        }
        stage('Deploy') {
            steps {
                script {
                    // Copy files to a web directory
                    sh 'mkdir -p /tmp/web-app'
                    sh 'cp *.html /tmp/web-app/'
                    
                    // Start a simple Python HTTP server on port 5000
                    sh '''
                        cd /tmp/web-app
                        python3 -m http.server 5000 > /dev/null 2>&1 &
                        echo $! > /tmp/web-server.pid
                    '''
                    
                    echo "Web application deployed at http://localhost:5000"
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    // Wait a moment for server to start
                    sh 'sleep 2'
                    
                    // Test the deployment
                    sh 'curl -f http://localhost:5000/index.html || exit 1'
                    echo "Application is running successfully!"
                }
            }
        }
    }
    post {
        always {
            script {
                // Optional cleanup (comment out if you want the server to keep running)
                // sh 'if [ -f /tmp/web-server.pid ]; then kill $(cat /tmp/web-server.pid) || true; fi'
                echo "Pipeline completed!"
            }
        }
    }
}