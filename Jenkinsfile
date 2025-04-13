pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                // Ambil kode dari GitHub
                git branch: 'main', url: 'https://github.com/suhadda12/demoweb.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                // Ganti sesuai kebutuhan
                // Contoh: Node.js
                sh 'npm install'
            }
        }

        stage('Run Unit Tests') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build') {
            steps {
                // Misal untuk Node.js
                sh 'npm run build'
            }
        }
    }

    post {
        success {
            echo '✅ Build & Test berhasil!'
            // Bisa ditambah notifikasi Slack/email
        }
        failure {
            echo '❌ Build/Test gagal!'
            // Bisa juga notifikasi di sini
        }
    }
}
