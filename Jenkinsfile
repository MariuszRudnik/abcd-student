pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('Prepare') {
            steps {
                sh 'mkdir -p results/'
            }
        }

        stage('DAST') {
            steps {
                sh '''
                    docker rm -f juice-shop || true
                    docker run --name juice-shop -d --rm \
                        -p 3000:3000 \
                        bkimminich/juice-shop
                '''
                sh 'sleep 15'
                sh '''
                    docker rm -f zap || true
                    docker run --name zap \
                    -v /Users/mariusz/Documents/DevSecOps/Test/.zap:/zap/wrk/:rw \
                    -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                    "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" || true
                '''
                sh 'sleep 15'
                sh 'docker logs zap'
            }
            post {
                always {
                    script {
                        try {
                            sh 'docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html'
                            sh 'docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml'
                        } catch (Exception e) {
                            echo 'Failed to copy reports from the zap container.'
                        }
                    }
                }
            }
        }

        stage('Cleanup') {
            steps {
                sh '''
                    docker rm -f zap juice-shop || true
                '''
            }
        }
    }
    post {
        always {
            echo 'Archiving results...'
            archiveArtifacts artifacts: 'results/**/*.html', fingerprint: true, allowEmptyArchive: true
            sh 'ls -la results/' // dodatkowa linia do sprawdzenia czy są jakieś pliki
            script {
                if (fileExists('results/zap_xml_report.xml')) {
                    echo 'Sending reports to DefectDojo...'
                    defectDojoPublisher(artifact: './results/zap_xml_report.xml',
                                        productName: 'Juice Shop',
                                        scanType: 'ZAP Scan',
                                        engagementName: 'mario360x@gmail.com')
                } else {
                    echo 'ZAP report not found, skipping DefectDojo publishing...'
                }
            }
        }
    }
}
