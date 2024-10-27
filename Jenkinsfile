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
                    mkdir -p ${WORKSPACE}/reports
                    chmod -R 777 ${WORKSPACE}/reports
                '''
                echo "Directory created. Waiting for 5 seconds..."
                sleep(5)
            }
        }

        stage('Step 4: Copy passive.yaml File') {
            steps {
                echo "Copying passive.yaml file to workspace directory for ZAP access..."
                sh '''
                    cp ${WORKSPACE}/passive.yaml ${WORKSPACE}/passive.yaml
                    chmod 777 ${WORKSPACE}/passive.yaml
                '''
                echo "File copied. Waiting for 5 seconds..."
                sleep(5)
            }
        }

        stage('Step 5: Run OWASP ZAP for Passive Scanning') {
            steps {
                echo "Starting OWASP ZAP container..."
                sh """
                    docker run --name zap \
                    --add-host=host.docker.internal:host-gateway \
                    -v ${WORKSPACE}/reports:/zap/wrk:rw \
                    -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                    "zap.sh -cmd -addonupdate; \
                    zap.sh -cmd -addoninstall communityScripts; \
                    zap.sh -cmd -addoninstall pscanrulesAlpha; \
                    zap.sh -cmd -addoninstall pscanrulesBeta; \
                    zap.sh -cmd -autorun /zap/wrk/passive.yaml" || true
                """
                echo "OWASP ZAP scan complete. Waiting for 5 seconds..."
                sleep(5)
            }
        }

        stage('Step 6: Verify and Archive Scan Results') {
            steps {
                echo "Verifying scan results..."
                sh 'ls -al ${WORKSPACE}/reports'

                echo "Archiving scan results..."
                archiveArtifacts artifacts: '${WORKSPACE}/reports/**/*', fingerprint: true, allowEmptyArchive: true
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
                if (fileExists("${WORKSPACE}/reports/zap_xml_report.xml")) {
                    echo "Sending ZAP XML report to DefectDojo..."
                    defectDojoPublisher(artifact: '${WORKSPACE}/reports/zap_xml_report.xml',
                                        productName: 'Juice Shop',
                                        scanType: 'ZAP Scan',
                                        engagementName: 'mario360x@gmail.com')
                } else {
                    echo "ZAP XML report not found, skipping DefectDojo upload."
                }
            }

            archiveArtifacts artifacts: '${WORKSPACE}/reports/**/*', fingerprint: true, allowEmptyArchive: true
        }
    }
}
