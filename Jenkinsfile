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
                    // Check and kill any processes on port 5002 (avoiding conflicts with port 5000)
                    sh '''
                        echo "Checking for processes using port 5002..."
                        # Try to kill any existing HTTP servers
                        pkill -f "python3 -m http.server 5002" || true
                        
                        # Force kill anything using port 5002
                        PORT=5002
                        PID=$(lsof -t -i:$PORT) || true
                        if [ ! -z "$PID" ]; then
                            echo "Found process $PID using port $PORT, killing it..."
                            kill -9 $PID || true
                        fi
                        sleep 2
                    '''
                    
                    // Clean up previous deployment
                    sh '''
                        rm -rf /tmp/web-app || true
                        rm -f /tmp/server.log || true
                        rm -f /tmp/web-server.pid || true
                    '''
                    
                    // Copy files to a web directory
                    sh 'mkdir -p /tmp/web-app'
                    sh 'cp *.html /tmp/web-app/'
                    
                    // Start a simple Python HTTP server on port 5002 in background
                    sh '''
                        cd /tmp/web-app
                        echo "Starting server on port 5002..."
                        # Use BUILD_ID=dontKillMe to prevent Jenkins from killing the process
                        BUILD_ID=dontKillMe nohup python3 -m http.server 5002 > /tmp/server.log 2>&1 &
                        echo $! > /tmp/web-server.pid
                        sleep 3
                        
                        # Verify server started correctly
                        if [ -f /tmp/server.log ]; then
                            echo "Server log exists, checking for errors..."
                            if grep -q "Error" /tmp/server.log; then
                                echo "Errors found in server log:"
                                cat /tmp/server.log
                            else
                                echo "No errors found in server log"
                            fi
                        fi
                    '''
                    
                    echo "Web application deployed at http://localhost:5002"
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
                        if ! ps aux | grep -q "[p]ython3 -m http.server 5002"; then
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
                            if curl -s -f -m 2 http://localhost:5002/ > /dev/null || curl -s -f -m 2 http://localhost:5002/index.html > /dev/null; then
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
                        netstat -an | grep 5002 || echo "No process listening on port 5002"
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
                echo "Pipeline completed! Cleaning up resources..."
                
                // Keep the server running for manual testing if successful
                if (currentBuild.result == 'SUCCESS') {
                    echo "Build successful - server is running at http://localhost:5002"
                } else {
                    // Clean up if the build failed
                    sh '''
                        echo "Cleaning up processes..."
                        if [ -f /tmp/web-server.pid ]; then
                            echo "Killing web server process..."
                            kill $(cat /tmp/web-server.pid) || true
                        fi
                        
                        # Make extra sure we kill any processes
                        pkill -f "python3 -m http.server 5002" || true
                        
                        # Check for any remaining processes on port 5002
                        PID=$(lsof -t -i:5002) || true
                        if [ ! -z "$PID" ]; then
                            echo "Found leftover process $PID on port 5002, killing it..."
                            kill -9 $PID || true
                        fi
                    '''
                }
            }
        }
        success {
            script {
                echo "✅ SUCCESS: Web application is available at http://localhost:5002"
            }
        }
        failure {
            script {
                echo "❌ FAILURE: Pipeline failed. See logs for details."
            }
        }
    }
}