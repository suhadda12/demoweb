pipeline {
    agent { label 'local-build' }

    environment {
        REPO_NAME = 'tester-web1'
        ZIP_FILE = "${REPO_NAME}.zip"
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/suhadda12/demoweb.git', branch: 'main'
            }
        }

        stage('Archive to Zip') {
            steps {
                sh '''
                    # Hapus ZIP lama jika ada
                    rm -f $ZIP_FILE

                    # Buat ZIP baru, exclude dirinya sendiri
                    zip -r $ZIP_FILE . -x $ZIP_FILE
                '''
            }
        }

        stage('Clean Up') {
            steps {
                sh 'rm -f $ZIP_FILE'
            }
        }
    }
}
