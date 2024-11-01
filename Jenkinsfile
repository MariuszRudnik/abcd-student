pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }

    stages {
        stage('Step 1: Code Checkout') {
            steps {
                script {
                    cleanWs()
                    echo "Cloning the GitHub repository..."
                    git credentialsId: 'github-pat', url: 'https://github.com/MariuszRudnik/abcd-student', branch: 'osv-scanner'
                    echo "Code cloned. Listing workspace contents..."
                    sh 'ls -al ${WORKSPACE}'
                    echo "Waiting for 5 seconds..."
                    sleep(5)
                }
            }
        }

        stage('Step 2: Prepare Juice Shop Application for Testing') {
            steps {
                script {
                    echo "Starting Juice Shop application..."
                    sh '''
                        docker run --name juice-shop -d --rm -p 3000:3000 bkimminich/juice-shop
                    '''
                    echo "Juice Shop is running. Waiting for 5 seconds..."
                    sleep(5)
                }
            }
        }

        stage('Step 3: Scan Juice Shop Container with OSV-Scanner') {
            steps {
                script {
                    echo "Copying package-lock.json from the juice-shop container..."
                    // Skopiuj plik package-lock.json z kontenera
                    sh 'docker cp juice-shop:/app/package-lock.json ./package-lock.json'

                    echo "Running OSV-Scanner on package-lock.json..."
                    // Uruchom OSV-Scanner i zapisz wynik w pliku
                    sh 'osv-scanner --lockfile=./package-lock.json > ./osv-scan-report.json'

                    echo "Checking if /Documents/DevSecOps/Test/osv directory exists..."
                    // Sprawdź, czy katalog istnieje, i utwórz go tylko w razie potrzeby
                    sh '''
                        if [ ! -d "/Documents/DevSecOps/Test/osv" ]; then
                            echo "Directory does not exist. Creating it..."
                            mkdir -p /Documents/DevSecOps/Test/osv
                        else
                            echo "Directory already exists. Skipping creation."
                        fi
                    '''

                    echo "Saving OSV scan report to /Documents/DevSecOps/Test/osv..."
                    // Przenieś raport do wskazanej lokalizacji
                    sh 'cp ./osv-scan-report.json /Documents/DevSecOps/Test/osv/osv-scan-report.json'

                    echo "OSV scan report saved successfully."
                }
            }
        }
    }
}
