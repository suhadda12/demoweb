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
                    // Format timestamp: YYYYMMDD_HHmmss
                    def timestamp = new Date().format("yyyyMMdd_HHmmss", TimeZone.getTimeZone('Asia/Jakarta'))
                    env.ZIP_FILE = "${env.REPO_NAME}_${timestamp}.zip"
                }

                sh '''
                    echo "Menghapus semua ZIP lama..."
                    rm -f ${REPO_NAME}_*.zip || true

                    echo "Membuat ZIP baru (tanpa .git)..."
                    zip -r $ZIP_FILE . -x ".git/*"
                '''
            }
        }
    }
}
