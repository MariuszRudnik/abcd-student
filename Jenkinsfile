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
                        if [ -f "/var/jenkins_home/workspace/ZAP/passive.yaml" ]; then
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

        stage('Step 3: Run ZAP Scanner') {
            steps {
                script {
                    echo "Starting ZAP container to run passive scan..."
                    sh '''
                        docker run --name zap \
                            --add-host=host.docker.internal:host-gateway \
                            -v "/var/jenkins_home/workspace/ZAP:/zap/wrk/:rw" \
                            -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                            "
                            if [ -f /zap/wrk/passive.yaml ]; then
                                echo 'passive.yaml found in container.';
                            else
                                echo 'passive.yaml NOT found in container.';
                                exit 1;
                            fi;
                            zap.sh -cmd -addonupdate; \
                            zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" || true
                    '''
                    echo "ZAP scan completed."
                }
            }
        }
            stage('Step 4: Copy Results') {
            steps {
                script {
                    echo "Copying ZAP results to result directory..."
                    sh '''
                        if [ ! -d "/var/jenkins_home/workspace/ZAP/result" ]; then
                            echo "Result directory does not exist. Creating result directory..."
                            mkdir -p /var/jenkins_home/workspace/ZAP/result
                        fi
                        find /var/jenkins_home/workspace/ZAP -mindepth 1 -maxdepth 1 ! -name 'result' -exec cp -r {} /var/jenkins_home/workspace/ZAP/result/ \;
                    '''
                    echo "Results copied successfully."
                }
            }
        }

}

    post {
        always {
            echo "Pipeline execution completed."
        }
    }
}
