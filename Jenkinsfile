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

        stage('Step 1.5: Generate package-lock.json using Docker') {
            steps {
                script {
                    echo "Running npm install in Docker to generate package-lock.json..."
                    sh '''
                        docker run --rm -v ${WORKSPACE}:/app -w /app node:latest npm install
                    '''
                    echo "package-lock.json generated successfully in the Jenkins workspace."
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
                    echo "Juice Shop is running. Waiting for 5 seconds..."
                    sleep(5)
                    
                    echo "Stopping Juice Shop container..."
                    sh 'docker stop juice-shop'
                    echo "Juice Shop container stopped."
                }
            }
        }

        stage('Step 3: Check for package-lock.json in Specified Directory') {
            steps {
                script {
                    echo "Checking if package-lock.json exists in the specified directory..."
                    sh '''
                        if [ -f "${WORKSPACE}/package-lock.json" ]; then
                            echo "package-lock.json exists in the specified directory."
                        else
                            echo "package-lock.json does NOT exist in the specified directory."
                            exit 1
                        fi
                    '''
                }
            }
        }

        stage('Step 4: Run OSV-Scanner on Host') {
            steps {
                script {
                    echo "Running OSV-Scanner on package-lock.json and saving results to results.txt..."
                    sh '''
                        osv-scanner scan --lockfile ~/Documents/DevSecOps/Test/workspace/osv-scanner/package-lock.json > ~/Documents/DevSecOps/Test/workspace/osv-scanner/results.txt
                    '''
                    echo "OSV-Scanner report generated at ~/Documents/DevSecOps/Test/workspace/osv-scanner/results.txt."
                }
            }
        }
    }
}
