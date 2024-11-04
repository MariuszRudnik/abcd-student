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
                    git credentialsId: 'github-pat', url: 'https://github.com/MariuszRudnik/abcd-student', branch: 'Semgrep'
                    echo "Code checked out. Listing workspace contents..."
                    sh 'ls -al ${WORKSPACE}'  // Wyświetlenie zawartości katalogu roboczego
                    echo "Waiting for 5 seconds..."
                    sleep(5) // Pauza 5 sekund
                }
            }
        }

        stage('Step 2: Verify rules.yaml exists') {
            steps {
                script {
                    echo "Checking if rules.yaml file exists in the workspace..."
                    if (fileExists('${WORKSPACE}/rules.yaml')) {
                        echo "rules.yaml file found."
                    } else {
                        echo "rules.yaml file not found."
                        error("rules.yaml is missing in the main directory.")
                    }
                }
            }
        }
    }
}
