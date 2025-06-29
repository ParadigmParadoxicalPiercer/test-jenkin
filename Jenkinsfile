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
                    // Kill any existing server on port 5000
                    sh 'pkill -f "python3 -m http.server 5000" || true'
                    sh 'sleep 1'
                    
                    // Copy files to a web directory
                    sh 'mkdir -p /tmp/web-app'
                    sh 'cp *.html /tmp/web-app/'
                    
                    // Start a simple Python HTTP server on port 5000 in background
                    sh '''
                        cd /tmp/web-app
                        nohup python3 -m http.server 5000 > /tmp/server.log 2>&1 &
                        echo $! > /tmp/web-server.pid
                        sleep 3
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
                    
                    // Test the deployment - try multiple approaches
                    sh '''
                        echo "Testing server status..."
                        if ps aux | grep -q "[p]ython3 -m http.server 5000"; then
                            echo "Server is running"
                        else
                            echo "Server not found"
                            exit 1
                        fi
                    '''
                    
                    // Test the actual file access
                    sh 'curl -f http://localhost:5000/ || curl -f http://localhost:5000/index.html'
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