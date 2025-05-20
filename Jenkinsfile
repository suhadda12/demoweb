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
                    echo "Membuat ZIP baru (tanpa .git dan Jenkinsfile)..."
                    zip -r $ZIP_FILE . -x ".git/*" "Jenkinsfile" ".workflow"
                '''
            }
        }

        stage('Deploy via Rsync (Direct SSH)') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'jenkins-to-apptest', keyFileVariable: 'SSH_KEY')]) {
                    sh '''
                        rsync -avzp -e "ssh -i $SSH_KEY -o StrictHostKeyChecking=yes" $ZIP_FILE ubuntu@192.168.137.111:/home/ubuntu/artifactory/
                    '''
                }
            }
        }

        stage('Upload to Artifactory via CURL') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'jfrog-cred1', usernameVariable: 'ART_USER', passwordVariable: 'ART_PASS')]) {
                    sh '''
                        echo "Upload file ke Artifactory..."
                        curl -u$ART_USER:$ART_PASS -T $ZIP_FILE http://192.168.137.111:8081/artifactory/tester-web1/$ZIP_FILE
                    '''
                }
            }
        }
        
        stage('Clean Up') {
            steps {
                sh 'rm -f $ZIP_FILE'
            }
        }
    }
}
