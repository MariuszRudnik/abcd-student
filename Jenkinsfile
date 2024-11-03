pipeline {
    agent any
    options {
        skipDefaultCheckout(true) // Pomijanie domyślnego checkoutu
    }

    stages {
        stage('Step 1: Code checkout for GitHub') {
            steps {
                script {
                    cleanWs() // Czyszczenie workspace
                    echo "Checking out code from GitHub repository..."
                    git credentialsId: 'github-pat', url: 'https://github.com/MariuszRudnik/abcd-student', branch: 'Trufflehog-Scan'
                    echo "Code checked out. Listing workspace contents..."
                    sh 'ls -al ${WORKSPACE}'  // Wyświetlenie zawartości katalogu roboczego
                    echo "Waiting for 5 seconds..."
                    sleep(5) // Pauza 5 sekund
                }
            }
        }

        stage('Step 2: Run Juice Shop Container') {
            steps {
                script {
                    echo "Starting Juice Shop container..."
                    sh '''
                        docker run --name juice-shop -d --rm -p 3000:3000 bkimminich/juice-shop
                    '''
                    echo "Juice Shop is running. Waiting for 20 seconds..."
                    sh 'trufflehog --rules /var/jenkins_home/workspace/TrufflehogScan/rules.yaml --json file:///var/jenkins_home/workspace/juice-shop > /var/jenkins_home/workspace/TrufflehogScan/scan_result.json'
                    sleep(20)
                    
                    echo "Stopping Juice Shop container..."
                    sh 'docker stop juice-shop'
                    echo "Juice Shop container stopped."
                }
            }
        }
    } // zamknięcie bloku stages i pipeline
}
