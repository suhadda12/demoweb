pipeline {
    agent { label 'local-build' }

    environment {
        REPO_NAME = 'tester-web1'
        ZIP_FILE = "${REPO_NAME}.zip"
        ARTIFACTORY_URL = 'http://192.168.137.111:8081/artifactory'
        TARGET_REPO = 'tester-web1'
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

        stage('Upload to JFrog') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'jenkins-artifactory', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh '''
                        curl -u "$USERNAME:$PASSWORD" -T "$ZIP_FILE" "$ARTIFACTORY_URL/$TARGET_REPO/$REPO_NAME/$ZIP_FILE"
                    '''
                }
            }
        }

        stage('Clean up') {
            steps {
                sh 'rm -f $ZIP_FILE'
            }
        }
    }
}
