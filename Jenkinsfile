
pipeline {
    agent any

    environment {
        ARTIFACTORY_URL = 'http://artifactory:8082/artifactory'
        REPO = 'jenkins-artifacts'
        VERSION = '1.0.0.3'
        GIT_REPO_NAME = 'Jenkins-hello-world'
    }

    stages {

        stage('Build Metadata') {
            steps {
                script {
                    // Generate build timestamp in IST
                    env.BUILD_TIMESTAMP = sh(
                        script: "TZ=Asia/Kolkata date '+%Y.%m.%d_%H.%M'",
                        returnStdout: true
                    ).trim()

                    // Get full and short commit hash
                    env.COMMIT_FULL = sh(
                        script: 'git rev-parse HEAD',
                        returnStdout: true
                    ).trim()

                    env.COMMIT_SHORT = env.COMMIT_FULL.take(7)

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

                    // Generate artifact names
                    env.ARTIFACT_NAME =
                        "${env.VERSION}_${env.BUILD_TYPE}_${env.BUILD_TIMESTAMP}_${env.COMMIT_SHORT}"

                    env.ZIP_NAME = "${env.ARTIFACT_NAME}.zip"

                    echo "Branch: ${env.BRANCH_NAME}"
                    echo "Build Type: ${env.BUILD_TYPE}"
                    echo "Full Commit: ${env.COMMIT_FULL}"
                    echo "Short Commit: ${env.COMMIT_SHORT}"
                    echo "Timestamp: ${env.BUILD_TIMESTAMP}"
                    echo "Artifact: ${env.ARTIFACT_NAME}"
                    echo "ZIP: ${env.ZIP_NAME}"
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

        stage('Prepare Artifact Package') {
            steps {
                sh '''
                    set -eu

                    # Create a clean packaging directory
                    rm -rf package
                    mkdir -p package

                    # Rename executable using artifact name
                    cp a.out "package/$ARTIFACT_NAME"

                    # Generate manifest.json
                    cat > package/manifest.json <<EOF
{
    "timestamp": "$BUILD_TIMESTAMP",
    "$GIT_REPO_NAME": {
        "commit": "$COMMIT_FULL"
    }
}
EOF

                    # Verify manifest was generated
                    test -s package/manifest.json

                    # Create ZIP containing exactly two files
                    cd package
                    jar --create --file "../$ZIP_NAME" "$ARTIFACT_NAME" manifest.json
                    cd ..

                    echo "ZIP package created: $ZIP_NAME"
                    echo "Package contents:"
                    jar --list --file "$ZIP_NAME"
                '''
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
                        set -eu

                        curl --fail --silent --show-error \
                          -u "$ART_USER:$ART_PASS" \
                          -T "$ZIP_NAME" \
                          "$ARTIFACTORY_URL/$REPO/$ARTIFACT_FOLDER/$ZIP_NAME"

                        echo "ZIP uploaded successfully!"
                    '''
                }
            }
        }

        stage('Success') {
            steps {
                echo "Build completed successfully!"
                echo "Uploaded ZIP: ${env.ARTIFACT_FOLDER}/${env.ZIP_NAME}"
                echo "Executable: ${env.ARTIFACT_NAME}"
                echo "Manifest: manifest.json"
                echo "Full Commit: ${env.COMMIT_FULL}"
            }
        }
    }

    post {
        always {
            echo "Jenkins build finished: ${currentBuild.currentResult}"
        }
    }
}
