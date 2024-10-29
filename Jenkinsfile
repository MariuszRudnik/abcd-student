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
                // Kopiowanie pliku passive.yaml do katalogu .zap, jeśli nie istnieje
                script {
                    def passiveFile = "${WORKSPACE}/.zap/passive.yaml"
                    if (!fileExists(passiveFile)) {
                        writeFile file: passiveFile, text: """
                          env:
                            contexts:
                              - name: test-context
                                urls:
                                  - "http://host.docker.internal:3000/"
                                includePaths:
                                  - "http://host.docker.internal:3000/.*"
                            parameters:
                              failOnError: true
                              failOnWarning: false
                              progressToStdout: true
                          jobs:
                            - type: alertFilter
                              alertFilters:
                                - ruleId: 10020
                                  newRisk: "Info"
                            - type: passiveScan-config
                              parameters:
                                maxAlertsPerRule: 10
                                scanOnlyInScope: true
                            - type: spider
                            - type: spiderAjax
                            - type: passiveScan-wait
                              parameters:
                                maxDuration: 5
                            - type: report
                              parameters:
                                template: traditional-html
                                reportDir: /zap/wrk/reports
                                reportFile: zap_html_report
                                reportTitle: "ZAP Passive Scan Report"
                            - type: report
                              parameters:
                                template: traditional-xml
                                reportDir: /zap/wrk/reports
                                reportFile: zap_xml_report
                                reportTitle: "ZAP Passive Scan Report"
                        """
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
