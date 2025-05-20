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
                    def timestamp = new Date().format("yyyyMMdd_HHmmss", TimeZone.getTimeZone('Asia/Jakarta'))
                    env.ZIP_FILE = "${env.REPO_NAME}_${timestamp}.zip"
                }

                sh '''
                    echo "Menghapus semua ZIP lama..."
                    rm -f ${REPO_NAME}_*.zip || true

                    echo "Membuat ZIP baru (tanpa .git dan Jenkinsfile)..."
                    zip -r $ZIP_FILE . -x ".git/*" "Jenkinsfile"
                '''
            }
        }

        stage('Deploy via Rsync (Direct SSH)') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'jenkins-to-apptest', keyFileVariable: 'SSH_KEY')]) {
                    sh '''
                        rsync -avzp -e "ssh -i $SSH_KEY -o StrictHostKeyChecking=no" $ZIP_FILE ubuntu@192.168.137.111:/home/ubuntu/artifactory/
                    '''
                }
            }
        }
    }
}
