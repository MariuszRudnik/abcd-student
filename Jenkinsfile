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
                        if [ -f "/var/jenkins_home/workspace/osv-scanner/package-lock.json" ]; then
                            echo "package-lock.json exists in the Jenkins workspace."
                        else
                            echo "package-lock.json does NOT exist in the Jenkins workspace."
                            exit 1
                        fi
                    '''
                }
            }
        }

        stage('Step 4: Run OSV Scanner') {
            steps {
                script {
                    echo "Running OSV Scanner on package-lock.json..."
                    sh '''
                        osv-scanner scan --lockfile /var/jenkins_home/workspace/osv-scanner/package-lock.json > /var/jenkins_home/workspace/osv-scanner/results.txt || true
                    '''
                    echo "OSV Scanner has finished scanning. Results saved to results.txt."
                }
            }
        }

        stage('Step 5: Check for Results File') {
            steps {
                script {
                    echo "Checking if results.txt exists in the Jenkins workspace..."
                    sh '''
                        if [ -f "/var/jenkins_home/workspace/osv-scanner/results.txt" ]; then
                            echo "results.txt exists in the Jenkins workspace."
                        else
                            echo "results.txt does NOT exist in the Jenkins workspace."
                            exit 1
                        fi
                    '''
                }
            }
        }
    }
}
