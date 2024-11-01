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
                    echo "Cloning the GitHub repository from branch ZAP..."
                    git credentialsId: 'github-pat', url: 'https://github.com/MariuszRudnik/abcd-student', branch: 'ZAP'
                    echo "Code cloned successfully."
                }
            }
        }

        stage('Step 1.5: Check for passive.yaml File') {
            steps {
                script {
                    echo "Checking if passive.yaml exists in the Jenkins workspace..."
                    sh '''
                        if [ -f "${WORKSPACE}/zap/passive.yaml" ]; then
                            echo "passive.yaml exists in the Jenkins workspace."
                        else
                            echo "passive.yaml does NOT exist in the Jenkins workspace."
                            exit 1
                        fi
                    '''
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
                    echo "Juice Shop is running. Waiting for 50 seconds..."
                    sleep(50)
                    
                    echo "Stopping Juice Shop container..."
                    sh 'docker stop juice-shop'
                    echo "Juice Shop container stopped."
                }
            }
        }
    }
}
