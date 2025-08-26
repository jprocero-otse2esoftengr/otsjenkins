 pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/jprocero-otse2esoftengr/otsjenkins.git'
            }
        }

        stage('Compile xUML') {
            steps {
                // Use bat for Windows
                bat '''
                    echo Compiling xUML model...
                    java -jar tools/xumlc-7.20.0.jar -i urlUrl.xml -o build/
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Build Succeeded'
        }
        failure {
            echo '❌ Build Failed - check logs'
        }
    }
}
