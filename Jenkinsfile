pipeline {
    agent any
    options {
        skipDefaultCheckout(true) // Pomijanie domyślnego checkoutu
    }

    stages {
        stage('Step 1: Code Checkout') {
            steps {
                script {
                    cleanWs() // Czyszczenie workspace
                    echo "Checking out code from GitHub repository..."
                    git credentialsId: 'github-pat', url: 'https://github.com/MariuszRudnik/abcd-student', branch: 'main'
                    echo "Code checked out. Listing workspace contents..."
                    sh 'ls -al ${WORKSPACE}'  // Wyświetlenie zawartości katalogu roboczego
                    echo "Waiting for 5 seconds..."
                    sleep(5) // Pauza 5 sekund
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
                    sleep(5) // Pauza 5 sekund
                }
            }
        }

        stage('Step 3: Prepare Directory for Scan Results') {
            steps {
                echo "Creating directory for scan results..."
                sh '''
                    mkdir -p /tmp/reports
                    chmod -R 777 /tmp/reports
                '''
                echo "Directory created. Waiting for 5 seconds..."
                sleep(5) // Pauza 5 sekund
            }
        }

        stage('Step 4: Copy passive.yaml File') {
            steps {
                echo "Copying passive.yaml file to /tmp directory for ZAP access..."
                // Kopiowanie pliku passive.yaml do /tmp
                sh '''
                    cp ${WORKSPACE}/passive.yaml /tmp/passive.yaml
                    chmod 777 /tmp/passive.yaml
                '''
                echo "File copied. Waiting for 5 seconds..."
                sleep(5) // Pauza 5 sekund
            }
        }

        stage('Step 5: Run OWASP ZAP for Passive Scanning') {
            steps {
                echo "Starting OWASP ZAP container and checking for passive.yaml file..."
                
                // Sprawdzenie, czy plik passive.yaml jest dostępny w /tmp
                sh 'ls -al /tmp/passive.yaml' // Wyświetla szczegóły pliku, jeśli jest obecny

                // Uruchomienie OWASP ZAP z nową ścieżką do passive.yaml
                sh '''
                    docker run --name zap \
                    --add-host=host.docker.internal:host-gateway \
                    -v /tmp:/zap/wrk:rw \
                    -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                    "ls -al /zap/wrk/passive.yaml; \  # Sprawdzanie dostępności passive.yaml w kontenerze
                    zap.sh -cmd -addonupdate; \
                    zap.sh -cmd -addoninstall communityScripts; \
                    zap.sh -cmd -addoninstall pscanrulesAlpha; \
                    zap.sh -cmd -addoninstall pscanrulesBeta; \
                    zap.sh -cmd -autorun /zap/wrk/passive.yaml" || true
                '''
                
                echo "OWASP ZAP scan complete. Waiting for 5 seconds..."
                sleep(5) // Pauza 5 sekund
            }
        }

        stage('Step 6: Verify and Archive Scan Results') {
            steps {
                echo "Verifying scan results in /tmp/reports..."
                sh 'ls -al /tmp/reports' // Sprawdzanie zawartości katalogu z wynikami

                echo "Checking /tmp directory for any results..."
                sh 'ls -al /tmp' // Sprawdzanie zawartości katalogu /tmp

                echo "Archiving scan results from /tmp/reports..."
                archiveArtifacts artifacts: '/tmp/**/*', fingerprint: true, allowEmptyArchive: true
                echo "Scan results archived. Waiting for 5 seconds..."
                sleep(5) // Pauza 5 sekund
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
                if (fileExists('/tmp/reports/zap_xml_report.xml')) {
                    echo "Sending ZAP XML report to DefectDojo..."
                    defectDojoPublisher(artifact: '/tmp/reports/zap_xml_report.xml',
                                        productName: 'Juice Shop',
                                        scanType: 'ZAP Scan',
                                        engagementName: 'mario360x@gmail.com')
                } else {
                    echo "ZAP XML report not found, skipping DefectDojo upload."
                }
            }

            // Archiwizowanie wyników w Jenkinsie
            archiveArtifacts artifacts: '/tmp/**/*', fingerprint: true, allowEmptyArchive: true
        }
    }
}
