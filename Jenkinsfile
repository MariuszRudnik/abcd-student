pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
        timeout(time: 20, unit: 'MINUTES')  // Dodano limit czasu, aby unikać wiecznego oczekiwania w razie problemów
    }
    stages {
        stage('Code checkout from GitHub') {
            steps {
                script {
                    cleanWs()  // Czyści workspace, aby uniknąć konfliktów z poprzednimi buildami
                    git credentialsId: 'github-pat', url: 'https://github.com/MariuszRudnik/abcd-student', branch: 'ZAP'
                }
            }
        }
        
        stage('Prepare') {
            steps {
                sh 'mkdir -p results/'  // Tworzy katalog na wyniki, jeśli go nie ma
            }
        }
        
        stage('DAST') {
            steps {
                script {
                    // Uruchamianie kontenera z Juice Shop
                    sh '''
                        docker run --name juice-shop -d --rm -p 3000:3000 bkimminich/juice-shop
                    '''
                    sleep 10  // Zwiększony czas oczekiwania, aby upewnić się, że kontener jest gotowy
                    
                    // Uruchamianie skanowania ZAP
                    sh '''
                        docker run --name zap \
                        -v ${WORKSPACE}/.zap:/zap/wrk/:rw \
                        -t ghcr.io/zaproxy/zaproxy:stable bash -c \
    "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" || true
                    '''
                }
            }
            post {
                always {
                    script {
                        // Kopiowanie raportów i zatrzymanie kontenerów
                        sh '''
                            docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html
                            docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
                            docker stop zap || true
                            docker stop juice-shop || true
                        '''
                    }
                }
            }
        }
    }
    post {
        always {
            echo 'Archiving results...'
            archiveArtifacts artifacts: 'results/**/*.html', fingerprint: true, allowEmptyArchive: true
            echo 'Sending reports to DefectDojo...'
            defectDojoPublisher(
                artifact: './results/zap_xml_report.xml',  // Poprawiona ścieżka
                productName: 'Juice Shop',
                scanType: 'ZAP Scan',
                engagementName: 'mario360x@gmail.com'
            )
        }
    }
}
