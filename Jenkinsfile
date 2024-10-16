pipeline {
    agent any

    environment {
        // Define environment variables
        ZAP_REPORTS_DIR = "${WORKSPACE}/zap-reports"
        ZAP_CONFIG_DIR = "${WORKSPACE}/zap-config"
    }

    stages {
        stage('Code checkout from GitHub') {
            steps {
                cleanWs() // Clean workspace
                git(
                    url: 'https://github.com/AdamKazmierczak/abcd-student',
                    branch: 'develop',
                    credentialsId: 'github-token'
                )
            }
        }

        stage('Prepare ZAP Scan') {
            steps {
                script {
                    // Create directories for ZAP reports and config
                    sh 'mkdir -p $ZAP_REPORTS_DIR'
                    sh 'mkdir -p $ZAP_CONFIG_DIR'
                }
            }
        }

        stage('ZAP Scan') {
            steps {
                script {
                    // Start the Juice Shop application in Docker
                    sh '''
                        docker run --name juice-shop -d --rm -p 3000:3000 bkimminich/juice-shop
                        sleep 5
                    '''

                    // Run the ZAP scan using the correct volume paths
                    sh '''
                        docker run --name zap --rm --add-host host.docker.internal:host-gateway \
                          -v "$ZAP_REPORTS_DIR:/zap/wrk/reports" \
                          -v "$ZAP_CONFIG_DIR:/zap/wrk" \
                          zaproxy/zap-stable bash -c \
                          "/zap/zap.sh -cmd -addonupdate && \
                           /zap/zap.sh -cmd -addoninstall communityScripts && \
                           /zap/zap.sh -cmd -addoninstall pscanrulesAlpha && \
                           /zap/zap.sh -cmd -addoninstall pscanrulesBeta && \
                           /zap/zap.sh -cmd -autorun /zap/wrk/passive.yaml"
                    '''
                }
            }
        }
    }

    post {
        always {
            script {
                // Stop the Juice Shop container
                sh 'docker stop juice-shop'

                // List the contents of the ZAP reports directory for debugging purposes
                sh 'ls -l $ZAP_REPORTS_DIR'

                // Archive the ZAP scan reports (HTML and XML)
                archiveArtifacts artifacts: 'zap-reports/*.html, zap-reports/*.xml', allowEmptyArchive: false

                // Publish the ZAP report to DefectDojo
                defectDojoPublisher defectDojoInstanceUrl: 'https://defectdojo-instance.com',
                    defectDojoApiKey: 'your-api-key',
                    artifactDirectory: 'zap-reports',
                    artifactType: 'zap',
                    artifact: 'zap-reports/zap_xml_report.xml'
            }
        }
        failure {
            echo 'Pipeline failed! Check the logs for errors.'
        }
    }
}
