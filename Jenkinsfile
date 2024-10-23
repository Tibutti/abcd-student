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
                    git credentialsId: 'github-token', url: 'https://github.com/Tibutti/abcd-student.git', branch: 'main'
                }
            }
        }
       stage('[ZAP] Baseline passive-scan') {
    steps {
        sh 'mkdir -p results/'
        sh '''
            docker run --name juice-shop -d --rm \
                -p 3000:3000 \
                bkimminich/juice-shop
            sleep 20
        '''
       sh '''
           docker run --user=root --name zap \
               --add-host=host.docker.internal:host-gateway \
               -v /mnt/c/Szkolenia/abcd-student/.zap/passive.yaml:/wrk:rw \
               -v /mnt/c/Szkolenia/Downloads/Reports/:/wrk/reports \
               -t ghcr.io/zaproxy/zaproxy:stable bash -c \
               
               "mkdir -p /wrk/reports && zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive_scan.yaml" \
               || true
      '''
    }
    post {
        always {
            sh '''
                cp zap:/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html
                cp zap:/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
                docker stop zap juice-shop
                docker rm zap
            '''
        }
    }
}
    }
}
