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

        stage('Step 3: Check for package-lock.json') {
            steps {
                script {
                    echo "Checking if package-lock.json exists in /Users/mariusz/Documents/DevSecOps/Test/workspace/osv-scanner..."
                    sh '''
                        if [ -f /Users/mariusz/Documents/DevSecOps/Test/workspace/osv-scanner/package-lock.json ]; then
                            echo "package-lock.json exists in the specified directory."
                        else
                            echo "package-lock.json does NOT exist in the specified directory."
                        fi
                    '''
                }
            }
        }
    }
}
