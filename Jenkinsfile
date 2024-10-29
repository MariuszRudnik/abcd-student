pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('Code checkout from GitHub') {
            steps {
                script {
                    cleanWs()
                    git credentialsId: 'devsecops', url: 'https://github.com/MariuszRudnik/abcd-student', branch: 'ZAP'
                }
            }
        }
        stage('Prepare') {
            steps {
                sh 'mkdir -p results/ .zap/' // Tworzenie katalogów na wyniki i konfigurację ZAP
                script {
                    // Ścieżka do pliku passive.yaml na komputerze lokalnym
                    def localPassiveFile = '/Users/mariusz/Documents/DevSecOps/Test/passive.yaml'
                    def workspacePassiveFile = "${WORKSPACE}/.zap/passive.yaml"
                    
                    // Sprawdzenie, czy plik istnieje w podanej ścieżce lokalnej
                    if (fileExists(localPassiveFile)) {
                        // Kopiowanie pliku passive.yaml z lokalnego katalogu do ${WORKSPACE}/.zap/
                        sh "cp ${localPassiveFile} ${workspacePassiveFile}"
                        echo "Plik passive.yaml skopiowany z lokalnej ścieżki do ${WORKSPACE}/.zap/"
                    } else {
                        error "Błąd: Plik passive.yaml nie został znaleziony w lokalnej ścieżce ${localPassiveFile}"
                    }
                }
            }
        }
        stage('Check for passive.yaml') {
            steps {
                script {
                    def passiveFileExists = fileExists("${WORKSPACE}/.zap/passive.yaml")
                    if (!passiveFileExists) {
                        error "Błąd: Plik passive.yaml nie został znaleziony w katalogu ${WORKSPACE}/.zap/"
                    } else {
                        echo "Plik passive.yaml został znaleziony."
                    }
                }
            }
        }
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
                    
                    // Debugowanie - sprawdzenie zawartości katalogu /zap/wrk/
                    // oraz uruchomienie ZAP z odpowiednią konfiguracją
                    sh '''
                        docker run --name zap \
                            --add-host=host.docker.internal:host-gateway \
                            -v "${WORKSPACE}/.zap:/zap/wrk/:rw" \
                            -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                            "ls -la /zap/wrk/; zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" || true
                    '''
                }
            }
            post {
                always {
                    script {
                        def reportGenerated = sh(
                            script: 'docker exec zap bash -c "[ -f /zap/wrk/reports/zap_html_report.html ] && [ -f /zap/wrk/reports/zap_xml_report.xml ]"',
                            returnStatus: true
                        ) == 0
                        
                        if (reportGenerated) {
                            sh '''
                                echo "Kopiowanie raportów z kontenera ZAP do workspace..."
                                docker cp zap:/zap/wrk/reports/zap_html_report.html "${WORKSPACE}/results/zap_html_report.html"
                                docker cp zap:/zap/wrk/reports/zap_xml_report.xml "${WORKSPACE}/results/zap_xml_report.xml"
                            '''
                        } else {
                            echo "Błąd: raporty ZAP nie zostały znalezione."
                        }
                        
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
