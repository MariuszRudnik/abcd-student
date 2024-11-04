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
                    sh 'ls -al /var/jenkins_home/workspace/Semgrep'  // Wyświetlenie zawartości katalogu roboczego
                    echo "Waiting for 5 seconds..."
                    sleep(5) // Pauza 5 sekund
                }
            }
        }

        stage('Step 2: Verify rules.yaml exists') {
            steps {
                script {
                    echo "Checking if rules.yaml file exists in the workspace..."
                    if (fileExists('/var/jenkins_home/workspace/Semgrep/rules.yaml')) {
                        echo "rules.yaml file found."
                    } else {
                        echo "rules.yaml file not found."
                        error("rules.yaml is missing in the main directory.")
                    }
                }
            }
        }

        stage('Step 3: Semgrep scan') {
            steps {
                script {
                    echo "Running Semgrep scan using rules.yaml..."
                    sh 'semgrep --config /var/jenkins_home/workspace/Semgrep/rules.yaml /var/jenkins_home/workspace/Semgrep --json > /var/jenkins_home/workspace/Semgrep/scan_results.json'
                    echo "Semgrep scan completed. Results saved to /var/jenkins_home/workspace/Semgrep/scan_results.json"
                }
            }
        }
    }
}
