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
                    git credentialsId: 'github-pat', url: 'https://github.com/MariuszRudnik/abcd-student', branch: 'main'
                    echo "Code cloned. Listing workspace contents..."
                    sh 'ls -al ${WORKSPACE}'
                    echo "Waiting for 5 seconds..."
                    sleep(5)
                }
            }
        }

        stage('Step 1.5: Generate package-lock.json') {
            steps {
                script {
                    echo "Installing dependencies to generate package-lock.json..."
                    // Wygenerowanie package-lock.json w workspace
                    sh 'npm install --prefix ~/Documents/DevSecOps/Test/workspace/osv-scanner@tmp'
                    echo "package-lock.json generated successfully."
                }
            }
        }

        stage('Step 2: Prepare Juice Shop Application for Testing') {
            steps {
                script {
                    echo "Checking if Juice Shop container is already running..."
                    // Usuń istniejący kontener, jeśli działa
                    sh '''
                        if [ "$(docker ps -q -f name=juice-shop)" ]; then
                            echo "Stopping and removing existing Juice Shop container..."
                            docker stop juice-shop
                        else
                            echo "No existing Juice Shop container running."
                        fi
                    '''

                    echo "Starting Juice Shop application..."
                    sh '''
                        docker run --name juice-shop -d --rm -p 3000:3000 bkimminich/juice-shop
                    '''
                    echo "Juice Shop is running. Waiting for 5 seconds..."
                    sleep(5)
                }
            }
        }

        stage('Step 3: Run OSV-Scanner on Host') {
            steps {
                script {
                    echo "Running OSV-Scanner on package-lock.json on host..."
                    // Uruchom OSV-Scanner na hoście, wskazując bezpośrednią ścieżkę do package-lock.json
                    sh '/usr/local/bin/osv-scanner --lockfile=~/Documents/DevSecOps/Test/workspace/osv-scanner@tmp/package-lock.json > ./osv-scan-report.json'

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
