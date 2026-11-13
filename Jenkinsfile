pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Code checked out from GitHub'
            }
        }

        stage('Check GCC') {
            steps {
                sh 'gcc --version'
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

        stage('Package Artifact') {
            steps {
                echo 'Packaging C executable'
                sh 'tar -czf hello-world-c.tar.gz hello'
                sh 'ls -lh hello-world-c.tar.gz'
            }
        }

        stage('Success') {
            steps {
                echo 'C Hello World Pipeline completed successfully!'
            }
        }
    }
}
