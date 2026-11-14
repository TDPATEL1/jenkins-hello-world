
pipeline {
    agent any

    environment {
        ARTIFACTORY_URL = 'http://artifactory:8082/artifactory'
        REPO = 'jenkins-artifacts'
    }

    stages {
        stage('Build Metadata') {
            steps {
                script {
                    env.BUILD_TIMESTAMP = sh(
                        script: 'date -u +%Y%m%d-%H%M%S',
                        returnStdout: true
                    ).trim()

                    env.ARTIFACT_NAME = "a.out-${env.BUILD_TIMESTAMP}"

                    if (env.CHANGE_ID) {
                        env.ARTIFACT_FOLDER = 'PR'
                    } else if (env.BRANCH_NAME == 'main') {
                        env.ARTIFACT_FOLDER = 'master'
                    } else {
                        env.ARTIFACT_FOLDER = env.BRANCH_NAME
                    }

                    echo "Branch: ${env.BRANCH_NAME}"
                    echo "Artifact: ${env.ARTIFACT_NAME}"
                    echo "Folder: ${env.ARTIFACT_FOLDER}"
                }
            }
        }

        stage('Check GCC') {
            steps {
                sh 'gcc --version'
            }
        }

        stage('Compile C Program') {
            steps {
                sh 'gcc hello.c -o a.out'
            }
        }

        stage('Run Hello World') {
            steps {
                sh './a.out'
            }
        }

        stage('Upload to Artifactory') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'artifactory-creds',
                        usernameVariable: 'ART_USER',
                        passwordVariable: 'ART_PASS'
                    )
                ]) {
                    sh '''
                        curl --fail --silent --show-error \
                          -u "$ART_USER:$ART_PASS" \
                          -T a.out \
                          "$ARTIFACTORY_URL/$REPO/$ARTIFACT_FOLDER/$ARTIFACT_NAME"
                    '''
                }
            }
        }

        stage('Success') {
            steps {
                echo "Build completed successfully!"
                echo "Uploaded: ${env.ARTIFACT_FOLDER}/${env.ARTIFACT_NAME}"
            }
        }
    }
}
