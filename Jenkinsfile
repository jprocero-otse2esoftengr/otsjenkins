pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/jprocero-otse2esoftengr/otsjenkins.git'
            }
        }

        stage('Compile xUML') {
            steps {
                sh '''
                echo "🚀 Compiling xUML model..."
                java -jar tools/xumlc-7.20.0.jar -clean -uml uml/urlUrl.xml
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Build Success - xUML compiled"
        }
        failure {
            echo "❌ Build Failed - check logs"
        }
    }
}
