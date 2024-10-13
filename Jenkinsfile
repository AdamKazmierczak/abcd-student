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
                    git credentialsId: 'github-pat', url: 'https://github.com/AdamKazmierczak/abcd-student', branch: 'develop'
                }
            }
        }
        stage('Example') {
            steps {
                // Add your actual steps here
                echo 'Running example stage'
            }
        }
        stage('SCA scan') {
            steps {
                sh 'osv-scanner scan --lockfile package-lock.json --format json --output results/sca-osv-scanner.json'
            }
        }
    }
    post {
        always {
            defectDojoPublisher(artifact: 'results/sca-osv-scanner.json', 
                productName: 'Juice Shop', 
                scanType: 'OSV Scan', 
                engagementName: 'krzysztof@bezpiecznykod.pl')
        }
    }
}
