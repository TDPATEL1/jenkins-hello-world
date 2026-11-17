
pipeline {
    agent any

    environment {
        ARTIFACTORY_URL = 'http://artifactory:8082/artifactory'
        REPO = 'jenkins-artifacts'
        VERSION = '1.0.0.3'
    }

    stages {
        stage('Build Metadata') {
            steps {
                script {
                    // Get build timestamp in Indian Standard Time
                    env.BUILD_TIMESTAMP = sh(
                        script: "TZ=Asia/Kolkata date '+%Y.%m.%d_%H.%M'",
                        returnStdout: true
                    ).trim()

                    // Get first seven characters of commit hash
                    env.COMMIT_SHORT = sh(
                        script: 'git rev-parse HEAD | cut -c1-7',
                        returnStdout: true
                    ).trim()

                    // Identify branch or Pull Request
                    if (env.CHANGE_ID) {
                        env.ARTIFACT_FOLDER = 'PR'
                        env.BUILD_TYPE = "PR-${env.CHANGE_ID}"
                    } else if (env.BRANCH_NAME == 'main') {
                        env.ARTIFACT_FOLDER = 'master'
                        env.BUILD_TYPE = 'master'
                    } else {
                        error("Unsupported branch: ${env.BRANCH_NAME}")
                    }

                    // Generate final artifact name
                    env.ARTIFACT_NAME =
                        "${env.VERSION}_${env.BUILD_TYPE}_${env.BUILD_TIMESTAMP}_${env.COMMIT_SHORT}"

                    echo "Branch: ${env.BRANCH_NAME}"
                    echo "Build Type: ${env.BUILD_TYPE}"
                    echo "Commit: ${env.COMMIT_SHORT}"
                    echo "Timestamp: ${env.BUILD_TIMESTAMP}"
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
