pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }

    stages {
        stage('Step 1: Code Checkout') {
            steps {
                script {
                    cleanWs()
                    echo "Cloning the GitHub repository from branch osv-scanner..."
                    git credentialsId: 'github-pat', url: 'https://github.com/MariuszRudnik/abcd-student', branch: 'osv-scanner'
                    echo "Code cloned successfully."
                }
            }
        }

        stage('Step 2: Run Juice Shop Container') {
            steps {
                script {
                    echo "Starting Juice Shop container..."
                    sh '''
                        docker run --name juice-shop -d --rm -p 3000:3000 bkimminich/juice-shop
                    '''
                    echo "Juice Shop is running. Waiting for 20 seconds..."
                    sleep(20)
                    
                    echo "Stopping Juice Shop container..."
                    sh 'docker stop juice-shop'
                    echo "Juice Shop container stopped."
                }
            }
        }

        stage('Step 3: Check for package-lock.json in Jenkins Workspace') {
            steps {
                script {
                    echo "Checking if package-lock.json exists in the Jenkins workspace..."
                    sh '''
                        if [ -f "${WORKSPACE}/package-lock.json" ]; then
                            echo "package-lock.json exists in the Jenkins workspace."
                        else
                            echo "package-lock.json does NOT exist in the Jenkins workspace."
                        fi
                    '''
                }
            }
        }

        stage('Step 4: Run osv-scanner in Docker Container') {
            steps {
                script {
                    echo "Running osv-scanner in Docker container..."
                    sh '''
                        # Debugging step to list files in the workspace
                        ls -l ${WORKSPACE}

                        # Run osv-scanner with appropriate arguments
                        docker run --rm -v ${WORKSPACE}:/workspace -w /workspace ghcr.io/google/osv-scanner:latest --version
                        docker run --rm -v ${WORKSPACE}:/workspace -w /workspace ghcr.io/google/osv-scanner:latest --json /workspace/package-lock.json > ${WORKSPACE}/osv_scan_results.json
                    '''
                }
            }
        }
    }
}
