pipeline {
    agent { label 'local-build' }

    environment {
        ZIP_FILE = 'build.zip'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build ZIP') {
            steps {
                sh '''
                    echo "Membuat ZIP: $ZIP_FILE"
                    zip -r $ZIP_FILE . -x ".git/*" "Jenkinsfile" ".workflow"
                '''
            }
        }

        stage('Selesai') {
            steps {
                echo "ZIP berhasil dibuat: ${env.ZIP_FILE}"
                sh 'ls -lh $ZIP_FILE'
            }
        }
    }
}
