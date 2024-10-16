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
          git credentialsId: 'github-token', url: 'https://github.com/AdamKazmierczak/abcd-student', branch: 'develop'
        }
      }
    }

    stage('ZAP Scan') {
      steps {
        // Run the Juice Shop application
        sh '''
          docker run --name juice-shop -d --rm \\
            -p 3000:3000 \\
            bkimminich/juice-shop
          sleep 5
        '''

        // Run the ZAP scan
        sh '''
          docker run --name zap --rm \\
            --add-host host.docker.internal:host-gateway \\
            -v '${WORKSPACE}/zap-reports:/zap/wrk/reports' \\
            -v '${WORKSPACE}/zap-config:/zap/wrk' \\
            zaproxy/zap-stable bash -c \\
            '/zap/zap.sh -cmd -addonupdate && \\
            /zap/zap.sh -cmd -addoninstall communityScripts && \\
            /zap/zap.sh -cmd -addoninstall pscanrulesAlpha && \\
            /zap/zap.sh -cmd -addoninstall pscanrulesBeta && \\
            /zap/zap.sh -cmd -autorun /zap/wrk/passive.yaml'
        '''
      }
    }
  }

  post {
    always {
      script {
        sh 'docker stop juice-shop || true'
      }

      // Debugging step to check the report directory
      script {
        sh 'ls -l ${WORKSPACE}/zap-reports/'
      }

      // Archive ZAP reports
      script {
        if (fileExists('zap-reports/zap_xml_report.xml')) {
          archiveArtifacts artifacts: 'zap-reports/zap_xml_report.xml', allowEmptyArchive: true
          archiveArtifacts artifacts: 'zap-reports/zap_html_report.html', allowEmptyArchive: true

          defectDojoPublisher(artifact: 'zap-reports/zap_xml_report.xml',
                              productName: 'Juice Shop',
                              scanType: 'ZAP Scan',
                              engagementName: 'adaml.kazmierczak@gmail.com')
        } else {
          echo 'ZAP report not found, skipping DefectDojo publishing.'
        }
      }
    }
  }
}
