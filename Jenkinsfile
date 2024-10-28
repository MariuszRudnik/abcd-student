pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('[ZAP] Baseline passive-scan') {
            steps {
                // Tworzenie katalogu na wyniki
                sh 'mkdir -p results/'

                // Sprawdzenie, czy plik passive.yaml istnieje w katalogu roboczym
                sh 'if [ -f ${WORKSPACE}/passive.yaml ]; then echo "Plik passive.yaml istnieje."; else echo "Brak pliku passive.yaml. Sprawdź konfigurację." && exit 1; fi'

                // Zatrzymanie i usunięcie istniejących kontenerów przed uruchomieniem nowych
                sh '''
                    docker stop juice-shop zap || true
                    docker rm juice-shop zap || true
                '''

                // Uruchomienie aplikacji Juice Shop
                sh '''
                    docker run --name juice-shop -d --rm \
                        -p 3000:3000 \
                        bkimminich/juice-shop
                    sleep 10
                '''

                // Uruchomienie skanu ZAP
                sh '''
                    docker run --name zap \
                        --add-host=host.docker.internal:host-gateway \
                        -v ${WORKSPACE}/passive.yaml:/zap/wrk/passive.yaml:rw \  // montowanie pliku zamiast katalogu
                        -v ${WORKSPACE}/results:/zap/wrk/reports:rw \
                        -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                        "zap.sh -cmd -addonupdate; \
                         zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta; \
                         zap.sh -cmd -autorun /zap/wrk/passive.yaml" || true
                '''

                // Sprawdzanie zawartości katalogu raportów w kontenerze ZAP
                sh 'docker exec zap ls -la /zap/wrk/reports || echo "Reports not generated."'
            }
            post {
                always {
                    // Kopiowanie raportów ZAP do lokalnego katalogu wyników
                    sh '''
                        docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html || true
                        docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml || true
                        
                        # Zatrzymanie i usunięcie kontenerów
                        docker stop zap juice-shop || true
                        docker rm zap juice-shop || true
                    '''
                }
            }
        }
    }
    post {
        always {
            script {
                echo 'Archiwizowanie wyników...'
                archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
            }
        }
    }
}
