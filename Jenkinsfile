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
        sh '''
           sh 'mkdir -p results'
          docker run --name juice-shop -d --rm \\
            -p 3000:3000 \\
            bkimminich/juice-shop
          sleep 5
         
        '''

        sh '''
      docker run --name zap --rm \\
        --add-host host.docker.internal:host-gateway \\
        -v '/mnt/c/git/abc/abcd-student/.zap:/zap/wrk:rw' \\
        -v '/mnt/c/git/abc/abcd-student-Reports/:/zap/wrk/reports' \\
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
            sh '''
              docker stop juice-shop || true
            '''
        }

        // Archive the report from the specified directory
        archiveArtifacts artifacts: '/results/zap_xml_report.xml', 
                         allowEmptyArchive: true

        // Publish to DefectDojo
        defectDojoPublisher(artifact: '/results/zap_xml_report.xml', 
                            productName: 'Juice Shop', 
                            scanType: 'ZAP Scan', 
                            engagementName: 'adaml.kazmierczak@gmail.com')
    }
}

    
}

