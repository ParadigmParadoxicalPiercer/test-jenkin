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
                        BUILD_ID=dontKillMe nohup python3 -m http.server 5000 > /tmp/server.log 2>&1 &
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
                    sh 'sleep 5'
                    
                    // Test the deployment with retry logic
                    sh '''
                        echo "Testing server status..."
                        # Check if the process is running
                        if ! ps aux | grep -q "[p]ython3 -m http.server 5000"; then
                            echo "ERROR: Server process not found"
                            # Try to see what happened
                            echo "--- Server Log ---"
                            cat /tmp/server.log || echo "No log file found"
                            echo "--- Process List ---"
                            ps aux | grep python || echo "No python processes"
                            exit 1
                        fi
                        
                        echo "Server process is running, testing connectivity..."
                        
                        # Try multiple times to connect
                        max_attempts=5
                        attempt=1
                        while [ $attempt -le $max_attempts ]; do
                            echo "Connection attempt $attempt of $max_attempts..."
                            if curl -s -f http://localhost:5000/ > /dev/null || curl -s -f http://localhost:5000/index.html > /dev/null; then
                                echo "SUCCESS: Connected to server!"
                                exit 0
                            else
                                echo "Connection attempt failed. Waiting to retry..."
                                sleep 2
                                attempt=$((attempt+1))
                            fi
                        done
                        
                        echo "ERROR: Failed to connect after $max_attempts attempts"
                        echo "--- Server Log ---"
                        cat /tmp/server.log || echo "No log file found"
                        netstat -an | grep 5000 || echo "No process listening on port 5000"
                        exit 1
                    '''
                    
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