pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('Stage 1: Code checkout from GitHub') {
            steps {
                script {
                    cleanWs()
                    git credentialsId: 'abcd', url: 'https://github.com/Tibutti/abcd', branch: 'main'  
                }
            }
        }

        stage(' Stage 2: Prepare') {
            steps {
                sh 'mkdir -p results/'
            }
        }

        stage('Stage 3: Skanowanie przy pomocy Semgrep') {
            steps {
                script {
                    sh 'semgrep scan --config auto --output results/semgrep-report.json --json'
                }
            }
            // post {
            //     always {
            //         defectDojoPublisher(artifact: 'results/semgrep-report.json', 
            //                 productName: 'Juice Shop', 
            //                 scanType: 'Semgrep JSON Report', 
            //                 engagementName: 'mateusz.tyburski81@gmail.com')
            //     }
            // }
        }

        stage('Stage 4: Skanowanie sekretów przez Trufflehog') {
            steps {
                sh 'trufflehog git file://. --only-verified --json > results/trufflehog.json'
            }
            // post {
            //     always {
            //         defectDojoPublisher(artifact: 'results/trufflehog.json', 
            //                 productName: 'Juice Shop', 
            //                 scanType: 'Trufflehog Scan', 
            //                 engagementName: 'mateusz.tyburski81@gmail.com')
            //     }
            // }
        }

        stage('Stage 5: Skanowanie za pomocą OSV scanner') {
             steps {
                 sh 'osv-scanner scan --lockfile package-lock.json --format json --output results/sca-osv-scanner.json || true'
             }
        //     post {
        //         always {
        //             defectDojoPublisher(artifact: 'results/sca-osv-scanner.json', 
        //                     productName: 'Juice Shop', 
        //                     scanType: 'OSV Scan', 
        //                     engagementName: 'mateusz.tyburski81@gmail.com')
        //         }
        //     }
        }

        stage('Stage 6: Skanowanie za pomocą DAST') {
            steps {
                sh '''
                    docker run --name juice-shop -d --rm \
                    -p 3000:3000 bkimminich/juice-shop
                    sleep 20
                '''
                sh '''
                    docker run --name zap \
                    -v /mnt/c/Szkolenia/abcd-student/.zap:/zap/wrk/:rw \
                    -t ghcr.io/zaproxy/zaproxy:stable \
                    bash -c "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" || true
                '''
            }
            post {
                always {
                    sh '''
                        docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html
                        docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
                        docker stop zap juice-shop
                        docker rm zap
                    '''
                }
            }
        }
    }
    
    // post {
    //     always {
    //         echo 'Archiwizowanie wyników...'
    //         archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
    //         echo 'Wysyłanie raportów do DefectDojo...'
    //         defectDojoPublisher(artifact: 'results/zap_xml_report.xml', productName: 'Juice Shop', scanType: 'ZAP Scan', engagementName: 'mateusz.tyburski81@gmail.com')
    //     }
    // }
}
