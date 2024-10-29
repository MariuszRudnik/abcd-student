pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    stages {
        // (pozostałe etapy bez zmian)
        
        stage('[ZAP] Baseline passive-scan') {
            steps {
                script {
                    // Uruchomienie Juice Shop
                    sh '''
                        docker run --name juice-shop -d --rm \
                            -p 3000:3000 bkimminich/juice-shop
                        echo "Juice Shop uruchomiony, czekam na pełne załadowanie..."
                        sleep 50
                    '''
                    
                    // Uruchomienie ZAP z odpowiednią konfiguracją
                    sh '''
                        docker run --name zap \
                            --add-host=host.docker.internal:host-gateway \
                            -v "${WORKSPACE}/.zap:/zap/wrk/:rw" \
                            -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                            "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" || true
                    '''
                }
            }
            post {
                always {
                    script {
                        // Kopiowanie raportów bez sprawdzania ich istnienia, ponieważ będą w kontenerze ZAP
                        sh '''
                            echo "Kopiowanie raportów z kontenera ZAP do workspace..."
                            docker cp zap:/zap/wrk/reports/zap_html_report.html "${WORKSPACE}/results/zap_html_report.html" || echo "Nie znaleziono zap_html_report.html"
                            docker cp zap:/zap/wrk/reports/zap_xml_report.xml "${WORKSPACE}/results/zap_xml_report.xml" || echo "Nie znaleziono zap_xml_report.xml"
                        '''
                        
                        // Zatrzymywanie i usuwanie kontenerów
                        sh '''
                            echo "Zatrzymywanie i usuwanie kontenerów..."
                            
                            if [ $(docker ps -q -f name=zap) ]; then
                                docker stop zap || true
                                docker rm zap || true
                            fi
                            
                            if [ $(docker ps -q -f name=juice-shop) ]; then
                                docker stop juice-shop || true
                                docker rm juice-shop || true
                            fi
                        '''
                    }
                }
            }
        }
    }
    post {
        always {
            echo 'Archiving results'
            archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
            echo 'Sending reports to DefectDojo'
            defectDojoPublisher(artifact: 'results/zap_xml_report.xml', productName: 'Juice Shop', scanType: 'ZAP Scan', engagementName: 'mario360x@gmail.com')
        }
    }
}
