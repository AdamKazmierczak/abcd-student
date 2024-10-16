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
          docker run --name juice-shop -d --rm \\
            -p 3000:3000 \\
            bkimminich/juice-shop
          sleep 5
        '''

        // sh '''
        //   docker run --name zap --rm \\
        //     --add-host host.docker.internal:host-gateway \\
        //     -v '/mnt/c/git/abc/abcd-student/.zap:/zap/wrk/:rw' \\
        //     -v '/mnt/c/git/abc/abcd-student-Reports/:/zap/wrk/reports' \\
        //     zaproxy/zap-stable bash -c \\
        //     'zap.sh -cmd -addonupdate && \\
        //     zap.sh -cmd -addoninstall communityScripts && \\
        //     zap.sh -cmd -addoninstall pscanrulesAlpha && \\
        //     zap.sh -cmd -addoninstall pscanrulesBeta && \\
        //     zap.sh -cmd -autorun /zap/wrk/passive.yaml'
        // '''
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

      defectDojoPublisher(
        artifact: '/tmp/abcd-student-Reports/zap_xml_report.xml',
        productName: 'Juice Shop',
        scanType: 'ZAP Scan',
        engagementName: 'adaml.kazmierczak@gmail.com'
      )
    }
  }
}
