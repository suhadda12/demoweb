pipeline {
    agent { label 'local-build' }

    environment {
        REPO_NAME = 'tester-web1'
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/suhadda12/demoweb.git', branch: 'main'
            }
        }

        stage('Archive to Zip with Timestamp') {
            steps {
                script {
                    // Format timestamp: YYYYMMDD_HHMMSS
                    def timestamp = new Date().format("yyyyMMdd_HHmmss", TimeZone.getTimeZone('Asia/Jakarta'))
                    env.ZIP_FILE = "${env.REPO_NAME}_${timestamp}.zip"
                }

                sh '''
                    echo "Nama file ZIP: $ZIP_FILE"

                    echo "Menghapus file ZIP lama (jika ada)..."
                    rm -f $ZIP_FILE

                    echo "Membuat file ZIP baru dengan timestamp..."
                    zip -r $ZIP_FILE . -x $ZIP_FILE
                '''
            }
        }
    }
}
