pipeline {
    agent { label 'local-build' }

    environment {
        REPO_NAME = 'tester-web1'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
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
                    zip -r $ZIP_FILE . -x ".git/*" "Jenkinsfile" ".workflow" || exit 1
                '''
            }
        }

        stage('Deploy via Rsync (Branch-based)') {
            steps {
                script {
                    def server = ""
                    if (env.BRANCH_NAME == "master") {
                        server = "ubuntu@192.168.137.111"
                    } else if (env.BRANCH_NAME.startsWith("test/")) {
                        server = "ubuntu@192.168.137.222"
                    } else {
                        error("Branch '${env.BRANCH_NAME}' tidak dikenali untuk rsync.")
                    }

                    withCredentials([sshUserPrivateKey(credentialsId: 'jenkins-to-apptest', keyFileVariable: 'SSH_KEY')]) {
                        sh """
                            echo "Deploying to server: ${server}"
                            rsync -avzp -e "ssh -i \$SSH_KEY -o StrictHostKeyChecking=yes" \$ZIP_FILE ${server}:/home/ubuntu/artifactory/
                        """
                    }
                }
            }
        }

        stage('Upload to Artifactory (Branch-based)') {
            steps {
                script {
                    def artifactoryPath = ""
                    if (env.BRANCH_NAME == "master") {
                        artifactoryPath = "master"
                    } else if (env.BRANCH_NAME.startsWith("test/")) {
                        artifactoryPath = "test"
                    } else {
                        error("Branch '${env.BRANCH_NAME}' tidak dikenali untuk upload ke Artifactory.")
                    }

                    withCredentials([usernamePassword(credentialsId: 'jfrog-cred1', usernameVariable: 'ART_USER', passwordVariable: 'ART_PASS')]) {
                        sh """
                            echo "Upload ke Artifactory path: ${artifactoryPath}"
                            curl -u\$ART_USER:\$ART_PASS -T \$ZIP_FILE http://192.168.137.111:8081/artifactory/${REPO_NAME}/${artifactoryPath}/\$ZIP_FILE
                        """
                    }
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
