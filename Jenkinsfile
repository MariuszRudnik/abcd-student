pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('Checkout') {
            steps {
                cleanWs()
                git credentialsId: 'github-pat', url: 'https://github.com/MariuszRudnik/abcd-student', branch: 'ZAP'
            }
        }
        stage('Prepare') {
            steps {
                sh 'mkdir -p results'
            }
        }
        stage('DAST') {
            steps {
                script {
                    echo 'Starting Juice Shop application...'
                    sh 'docker run --name juice-shop -d --rm -p 3000:3000 bkimminich/juice-shop'
                    
                    echo 'Running ZAP scan...'
                    sh '''
            docker run --name zap \
                --add-host=host.docker.internal:host-gateway \
                -v /path/to/dir/with/passive/scan/yaml:/zap/wrk/:rw
                -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive_scan.yaml" \
                || true
        '''
                    
                    echo 'Verifying if ZAP container is running...'
                    sh 'docker ps | grep zap || echo "ZAP container is not running."'
                    
                    echo 'Checking ZAP results directory content...'
                    sh 'ls -la ${WORKSPACE}/results'
                }
            }
        }
    }
    post {
        always {
            script {
                echo 'Copying ZAP reports...'
                sh '''
                    docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html || true
                    docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml || true
                '''
                
                echo 'Stopping and cleaning up Docker containers...'
                sh '''
                    docker stop zap juice-shop || true
                    docker rm zap juice-shop || true
                '''
                
                echo 'Checking for ZAP XML report and uploading to DefectDojo...'
                if (fileExists('${WORKSPACE}/results/zap_xml_report.xml')) {
                    defectDojoPublisher(
                        artifact: '${WORKSPACE}/results/zap_xml_report.xml',
                        productName: 'Juice Shop',
                        scanType: 'ZAP Scan',
                        engagementName: 'mario360x@gmail.com'
                    )
                } else {
                    echo 'ZAP XML report not found, skipping DefectDojo upload.'
                }
                
                echo 'Archiving artifacts...'
                archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
            }
        }
    }
}
