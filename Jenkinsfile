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
                sh 'zip -r $ZIP_FILE .'
            }
        }
    }
}
