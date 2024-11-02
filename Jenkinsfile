pipeline {
    agent any
    
    options {
        // Opcja, aby pominąć domyślny checkout kodu, ponieważ wykonujemy własny
        skipDefaultCheckout(true)
    }

    stages {
        stage('Step 1: Code Checkout') {
            steps {
                script {
                    // Czyścimy przestrzeń roboczą, aby upewnić się, że nie ma pozostałości po poprzednich buildach
                    cleanWs()
                    
                    // Wyświetlamy komunikat, aby poinformować, że rozpoczynamy klonowanie repozytorium
                    echo "Cloning the GitHub repository from branch ZAP..."
                    
                    // Klonujemy repozytorium z określoną gałęzią (branch 'ZAP') z użyciem poświadczeń (credentialsId)
                    git credentialsId: 'github-pat', url: 'https://github.com/MariuszRudnik/abcd-student', branch: 'ZAP'
                    
                    // Potwierdzamy, że kod został sklonowany
                    echo "Code cloned successfully."
                }
            }
        }
        
        stage('Prepare report space') {
            steps {
                // Tworzymy katalog 'results/', aby przechowywać wyniki
                sh 'mkdir -p results/'
            }
        }
        
        stage('Step 2: Start Juice Shop Container') {
            steps {
                script {
                    // Uruchamiamy kontener Juice Shop
                    echo "Starting Juice Shop container..."
                    sh '''
                        docker run --name juice-shop -d --rm -p 3000:3000 bkimminich/juice-shop
                    '''
                    
                    // Czekamy 50 sekund, aby kontener miał czas na pełne uruchomienie
                    echo "Juice Shop is running. Waiting for 50 seconds..."
                    sleep(50)
                    
                    // Uruchamiamy kontener ZAP, gdy Juice Shop działa
                    echo "Starting ZAP container to perform security scans..."
                    sh '''
                        docker run --name zap \
                            --add-host=host.docker.internal:host-gateway \
                            -v ~/Documents/DevSecOps/Test/workspace/ZAP:/zap/wrk/:rw \
                            -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                            "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" || true
                    '''
                    
                    // Zatrzymujemy kontener Juice Shop
                    echo "Stopping Juice Shop container..."
                    sh 'docker stop juice-shop'
                    echo "Juice Shop container stopped."
                }
            }
        }
        
        stage('Step 3: Verify ZAP Report') {
            steps {
                script {
                    // Sprawdzamy, czy w katalogu 'results/' znajduje się plik wygenerowany przez ZAP
                    echo "Verifying if ZAP report exists in the 'results/' directory..."
                    def zapReportExists = sh(script: 'test -f results/passive.yaml', returnStatus: true) == 0
                    
                    if (zapReportExists) {
                        echo "ZAP report found. Verification successful."
                    } else {
                        error "ZAP report not found in the 'results/' directory. Verification failed."
                    }
                }
            }
        }
    }
}


