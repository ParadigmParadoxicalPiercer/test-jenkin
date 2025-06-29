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
                        
                        # Try a range of ports in case one is blocked
                        for PORT in 5002 5003 5004 5005 8080; do
                            echo "Trying port $PORT..."
                            
                            # Check if port is already in use
                            if lsof -i:$PORT > /dev/null; then
                                echo "Port $PORT is already in use, trying next port..."
                                continue
                            fi
                            
                            # Use BUILD_ID=dontKillMe to prevent Jenkins from killing the process
                            BUILD_ID=dontKillMe nohup python3 -m http.server $PORT > /tmp/server.log 2>&1 &
                            SERVER_PID=$!
                            echo $SERVER_PID > /tmp/web-server.pid
                            echo "Server started with PID $SERVER_PID on port $PORT"
                            
                            # Save the port for later use
                            echo $PORT > /tmp/server.port
                            
                            # Wait to ensure server is still running
                            sleep 3
                            
                            # Check if process is still running
                            if ps -p $SERVER_PID > /dev/null; then
                                echo "Server process confirmed running on port $PORT"
                                break
                            else
                                echo "Server failed to start on port $PORT"
                                cat /tmp/server.log
                            fi
                        done
                        
                        # Confirm server port and status
                        SERVER_PORT=$(cat /tmp/server.port || echo "5002")
                        echo "Using port: $SERVER_PORT"
                        
                        # Verify server started correctly
                        if [ -f /tmp/server.log ]; then
                            echo "Server log exists, checking for errors..."
                            if grep -q "Error\|Traceback" /tmp/server.log; then
                                echo "Errors found in server log:"
                                cat /tmp/server.log
                            else
                                echo "No errors found in server log"
                            fi
                        fi
                    '''
                    
                    // Get the port that was actually used
                    env.SERVER_PORT = sh(script: "cat /tmp/server.port || echo '5002'", returnStdout: true).trim()
                    echo "Web application deployed at http://localhost:${env.SERVER_PORT}"
                    
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
                        # Get the port the server is using
                        SERVER_PORT=$(cat /tmp/server.port || echo "5002")
                        echo "Testing server status on port $SERVER_PORT..."
                        
                        # Check if the process is running
                        if [ -f /tmp/web-server.pid ]; then
                            PID=$(cat /tmp/web-server.pid)
                            echo "Server PID: $PID"
                            if ! ps -p $PID > /dev/null; then
                                echo "ERROR: Server process $PID not found"
                                # Try to see what happened
                                echo "--- Server Log ---"
                                cat /tmp/server.log || echo "No log file found"
                                echo "--- Process List ---"
                                ps aux | grep python || echo "No python processes"
                                exit 1
                            else
                                echo "Server process $PID is running"
                            fi
                        else
                            echo "ERROR: No PID file found"
                            exit 1
                        fi
                        
                        echo "Server process is running, testing connectivity on port $SERVER_PORT..."
                        
                        # Try multiple times to connect
                        max_attempts=10
                        attempt=1
                        while [ $attempt -le $max_attempts ]; do
                            echo "Connection attempt $attempt of $max_attempts..."
                            if curl -s -f -m 5 http://localhost:$SERVER_PORT/ > /dev/null || curl -s -f -m 5 http://localhost:$SERVER_PORT/index.html > /dev/null; then
                                echo "SUCCESS: Connected to server!"
                                exit 0
                            else
                                echo "Connection attempt failed. Waiting to retry..."
                                sleep 3
                                attempt=$((attempt+1))
                            fi
                        done
                        
                        echo "ERROR: Failed to connect after $max_attempts attempts"
                        echo "--- Server Log ---"
                        cat /tmp/server.log || echo "No log file found"
                        echo "--- Network Status ---"
                        lsof -i:$SERVER_PORT || echo "No process listening on port $SERVER_PORT"
                        netstat -an | grep $SERVER_PORT || echo "No netstat entries for port $SERVER_PORT"
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
                
                // Get the server port
                def serverPort = sh(script: "cat /tmp/server.port || echo '5002'", returnStdout: true).trim()
                
                // Keep the server running for manual testing if successful
                if (currentBuild.result == 'SUCCESS' || currentBuild.resultIsBetterOrEqualTo('SUCCESS')) {
                    echo "Build successful - server is running at http://localhost:${serverPort}"
                } else {
                    // Clean up if the build failed
                    sh '''
                        echo "Cleaning up processes..."
                        if [ -f /tmp/web-server.pid ]; then
                            echo "Killing web server process..."
                            PID=$(cat /tmp/web-server.pid)
                            echo "Killing process $PID"
                            kill $PID || true
                            kill -9 $PID || true
                        fi
                        
                        # Get the port that was used
                        SERVER_PORT=$(cat /tmp/server.port || echo "5002")
                        echo "Cleaning up port $SERVER_PORT"
                        
                        # Make extra sure we kill any HTTP server processes
                        pkill -f "python3 -m http.server" || true
                        
                        # Check for any remaining processes on the port
                        PID=$(lsof -t -i:$SERVER_PORT) || true
                        if [ ! -z "$PID" ]; then
                            echo "Found leftover process $PID on port $SERVER_PORT, killing it..."
                            kill -9 $PID || true
                        fi
                    '''
                }
            }
        }
        success {
            script {
                def serverPort = sh(script: "cat /tmp/server.port || echo '5002'", returnStdout: true).trim()
                echo "✅ SUCCESS: Web application is available at http://localhost:${serverPort}"
            }
        }
        failure {
            script {
                echo "❌ FAILURE: Pipeline failed. See logs for details."
            }
        }
    }
}