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
                    echo "Checking out code from GitHub repository..."
                    git credentialsId: 'github-pat', url: 'https://github.com/MariuszRudnik/abcd-student', branch: 'main'
                    echo "Code checked out. Listing workspace contents..."
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

        stage('Step 3: Prepare Directory for Scan Results') {
            steps {
                echo "Creating directory for scan results..."
                sh '''
                    mkdir -p /zap-results/reports
                    chmod -R 777 /zap-results/reports
                '''
                echo "Directory created. Waiting for 5 seconds..."
                sleep(5)
            }
        }

        stage('Step 4: Copy passive.yaml File') {
            steps {
                echo "Copying passive.yaml file from repository to workspace..."
                sh '''
                    cp ${WORKSPACE}/passive_scan.yaml /zap-results/passive_scan.yaml
                '''
                echo "File copied. Waiting for 5 seconds..."
                sleep(5)
            }
        }

        stage('Step 5: Run OWASP ZAP for Passive Scanning') {
            steps {
                echo "Starting OWASP ZAP container with full paths..."
                sh '''
                    docker run --name zap \
                    -v /zap-results:/zap/wrk/:rw \
                    -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                    "zap.sh -cmd -addonupdate; \
                    zap.sh -cmd -addoninstall communityScripts; \
                    zap.sh -cmd -addoninstall pscanrulesAlpha; \
                    zap.sh -cmd -addoninstall pscanrulesBeta; \
                    zap.sh -cmd -autorun /zap/wrk/passive_scan.yaml" || true
                '''
                echo "Listing contents of /zap-results directory..."
                sh 'ls -al /zap-results/'
                echo "Listing contents of /zap-results/reports directory..."
                sh 'ls -al /zap-results/reports/'
                
                echo "Fetching ZAP container logs..."
                sh 'docker logs zap'
                
                echo "OWASP ZAP scan complete. Waiting for 5 seconds..."
                sleep(5)
            }
        }

        stage('Step 6: Copy Reports to Jenkins Workspace') {
            steps {
                echo "Copying ZAP reports from ZAP container to Jenkins workspace..."
                sh '''
                    docker cp zap:/zap/wrk/reports ${WORKSPACE}/zap-reports
                '''
                echo "Reports copied. Listing contents of zap-reports directory..."
                sh 'ls -al ${WORKSPACE}/zap-reports/'
            }
        }

        stage('Step 7: Verify and Archive Scan Results') {
            steps {
                echo "Verifying scan results..."
                sh '''
                    ls -al ${WORKSPACE}/zap-reports/
                '''
                echo "Archiving scan results..."
                archiveArtifacts artifacts: 'zap-reports/**/*', fingerprint: true, allowEmptyArchive: true
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
                    docker stop zap juice-shop || true
                    docker rm zap juice-shop || true
                '''
                echo "Containers stopped and removed."

                echo "Checking if ZAP XML report exists..."
                if (fileExists('${WORKSPACE}/zap-reports/zap_xml_report.xml')) {
                    echo "Sending ZAP XML report to DefectDojo..."
                    defectDojoPublisher(artifact: '${WORKSPACE}/zap-reports/zap_xml_report.xml',
                                        productName: 'Juice Shop',
                                        scanType: 'ZAP Scan',
                                        engagementName: 'mario360x@gmail.com')
                } else {
                    echo "ZAP XML report not found, skipping DefectDojo upload."
                }
            }

            archiveArtifacts artifacts: 'zap-reports/**/*', fingerprint: true, allowEmptyArchive: true
        }
    }
}
