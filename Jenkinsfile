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
                    git credentialsId: 'github-token', url: 'https://github.com/Tibutti/abcd-student', branch: 'main' 
                }
            }
        }

        stage('Prepare') {
            steps {
                sh 'mkdir -p results/'
            }
        }

        stage('Skanowanie sekretów przez Trufflehog') {
             steps {
                 sh 'trufflehog git file://. --only-verified --json > results/trufflehog.json'
             }
             post {
                 always {
                     defectDojoPublisher(artifact: 'results/trufflehog.json', 
                             productName: 'Juice Shop', 
                             scanType: 'Trufflehog Scan', 
                             engagementName: 'mateusz.tyburski81@gmail.com')
                     
                 }
             }
        }
        // stage('SCA scan - przy pomocy osv-scanner ') {
        //    environment {
        //        RESULT_PATH = 'results/sca-osv-scanner.json'
        //        PRODUCT_NAME = 'Juice Shop'
        //        ENGAGEMENT_NAME = 'mateusz.tyburski81@gmail.com'
        //    }

        //    steps {
        //        // Uruchomienie osv-scanner i sprawdzenie sukcesu
        //        script {
        //            def scanResult = sh(script: 'osv-scanner scan --lockfile package-lock.json --format json --output ${RESULT_PATH}', returnStatus: true)
        //            if (scanResult != 0) {
        //                echo 'Skanowanie SCA nie powiodło się, sprawdź wynik polecenia dla szczegółów.'
        //            } else {
        //                echo 'Skanowanie SCA zakończone pomyślnie.'
        //            }

        //            // Sprawdzenie, czy plik wynikowy istnieje przed publikacją
        //            if (fileExists(RESULT_PATH)) {
        //                defectDojoPublisher(
        //                    artifact: RESULT_PATH, 
        //                    productName: PRODUCT_NAME, 
        //                    scanType: 'OSV Scan', 
        //                    engagementName: ENGAGEMENT_NAME
        //                )
        //            } else {
        //                echo "Artefakt ${RESULT_PATH} nie istnieje, pomijam publikację w DefectDojo."
        //            }
        //        }
        //   }
        // }

        // Zakomentowany etap "DAST"
        //
        // stage('DAST') {
        //     steps {
        //         sh '''
        //             docker run --name juice-shop -d --rm \
        //             -p 3000:3000 bkimminich/juice-shop
        //             sleep 20
        //         '''
        //         sh '''
        //             docker run --name zap \
        //             -v /mnt/c/Szkolenia/abcd-student/.zap:/zap/wrk/:rw \
        //             -t ghcr.io/zaproxy/zaproxy:stable \
        //             bash -c "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" || true
        //         '''
        //     }
        //     post {
        //         always {
        //             sh '''
        //                 docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html
        //                 docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
        //                 docker stop zap juice-shop
        //                 docker rm zap
        //             '''
        //         }
        //     }
        // }

    }
    
    // Zakomentowany blok post-pipeline związany z DAST
    //
    // post {
    //     always {
    //         echo 'Archiwizowanie wyników...'
    //         archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
    //         echo 'Wysyłanie raportów do DefectDojo...'
    //         defectDojoPublisher(artifact: 'results/zap_xml_report.xml', productName: 'Juice Shop', scanType: 'ZAP Scan', engagementName: 'mateusz.tyburski81@gmail.com')
    //     }
    // }
}
