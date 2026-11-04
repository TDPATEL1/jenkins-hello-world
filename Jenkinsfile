pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Code checked out from GitHub'
            }
        }

        stage('Install GCC') {
            steps {
                sh '''
                    apt-get update
                    apt-get install -y build-essential
                '''
            }
        }

        stage('Compile C Program') {
            steps {
                sh 'gcc hello.c -o hello'
            }
        }

        stage('Run Hello World') {
            steps {
                sh './hello'
            }
        }

        stage('Success') {
            steps {
                echo 'C Hello World Pipeline completed successfully!'
            }
        }
    }
}
