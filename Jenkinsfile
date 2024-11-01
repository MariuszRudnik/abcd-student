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

        stage('Step 4: Run osv-scanner and Check Version') {
            steps {
                script {
                    echo "Running osv-scanner to check version..."
                    sh '''
                        export PATH=$PATH:/go/bin

                        if ! command -v osv-scanner &> /dev/null
                        then
                            echo "osv-scanner could not be found, please install it first."
                            exit 1
                        fi

                        echo "osv-scanner version:"
                        osv-scanner --version
                    '''
                }
            }
        }
    }
}
