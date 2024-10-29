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
                    git credentialsId: 'github-pat', url: 'https://github.com/MariuszRudnik/abcd-student', branch: 'ZAP'
                }
            }
        }
        
        stage('Prepare') {
            steps {
                sh 'mkdir -p results/'
            }
        }
        
        stage('DAST') {
            steps {
               sh '''
                    docker run --name juice-shop -d --rm \
                        -p 3000:3000 \
                        bkimminich/juice-shop
                    sleep 5
                '''
                sh '''
                    docker run --name zap \
                    -v /Users/mariusz/Documents/DevSecOps/Test/.zap:/zap/wrk/:rw \
                     -t ghcr.io/zaproxy/zaproxy:stable bash -c \
    "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" || true

                '''
            }
            post {
                always {
                    sh '''
                        docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html
                        docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
                    '''
                }
            }
        }
        
        stage('Cleanup') {
            steps {
                sh '''
                    docker stop zap juice-shop
                    docker rm zap juice-shop
                '''
            }
        }
    }
}