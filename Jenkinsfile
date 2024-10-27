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
                    echo "Cloning the GitHub repository..."
                    git credentialsId: 'github-pat', url: 'https://github.com/MariuszRudnik/abcd-student', branch: 'main'
                    echo "Code cloned. Listing workspace contents..."
                    sh 'ls -al ${WORKSPACE}'
                    echo "Waiting for 5 seconds..."
                    sleep(5)
                }
            }
        }

        stage('Step 2: Prepare Juice Shop Application for Testing') {
            steps {
                script {
                    echo "Starting Juice Shop application..."
                    sh '''
                        docker run --name juice-shop -d --rm -p 3000:3000 bkimminich/juice-shop
                    '''
                    echo "Juice Shop is running. Waiting for 5 seconds..."
                    sleep(5)
                }
            }
        }

        stage('Step 3: Run OSV-Scanner for Vulnerability Scanning') {
            steps {
                script {
                    echo "Running OSV-Scanner as a Docker container..."
                    sh '''
                        docker run --rm -v ${WORKSPACE}:/workspace:ro gcr.io/osv-scanner/osv-scanner --output osv_scan_report.json --path /workspace
                    '''
                    echo "OSV-Scanner scan completed. Waiting for 5 seconds..."
                    sleep(5)
                }
            }
        }

        stage('Step 4: Verify and Archive Scan Results') {
            steps {
                echo "Verifying OSV-Scanner scan results..."
                sh 'ls -al ${WORKSPACE}'
                echo "Archiving OSV-Scanner scan results..."
                archiveArtifacts artifacts: 'osv_scan_report.json', fingerprint: true, allowEmptyArchive: true
                echo "Scan results archived. Waiting for 5 seconds..."
                sleep(5)
            }
        }
    }

    post {
        always {
            script {
                echo "Cleaning up Docker containers..."
                sh '''
                    docker stop juice-shop || true
                    docker rm juice-shop || true
                '''
                echo "Containers stopped and removed."

                echo "Checking if OSV-Scanner report exists..."
                if (fileExists('${WORKSPACE}/osv_scan_report.json')) {
                    echo "Sending OSV-Scanner report to DefectDojo..."
                    defectDojoPublisher(artifact: '${WORKSPACE}/osv_scan_report.json',
                                        productName: 'Juice Shop',
                                        scanType: 'OSV-Scan',
                                        engagementName: 'mario360x@gmail.com')
                } else {
                    echo "OSV-Scanner report not found, skipping DefectDojo upload."
                }
            }

            archiveArtifacts artifacts: '**/*', fingerprint: true, allowEmptyArchive: true
        }
    }
}
