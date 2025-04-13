pipeline {
    agent any  // Menggunakan agen Jenkins mana saja yang tersedia

    stages {
        stage('Clone Repository') {
            steps {
                // Melakukan clone repository dari GitHub
                git branch: 'main', url: 'https://github.com/suhadda12/demoweb.git'
            }
        }
    }
}
