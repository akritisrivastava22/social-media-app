pipeline {
    agent any

    tools {
        jdk 'JDK17'
        maven 'Maven'
    }

    environment {
        JFROG_URL = 'https://devopscg.jfrog.io/artifactory/springboot-war-repo'
    }

    options {
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    // Get the branch name from the webhook
                    def branch = env.BRANCH_NAME ?: 'main'
                    echo "Building branch: ${branch}"
                    
                    // Remove any existing temp directory
                    sh 'rm -rf temp-repo'
                    
                    // Clone the repo and checkout the branch
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'github-creds',
                            usernameVariable: 'GIT_USER',
                            passwordVariable: 'GIT_TOKEN'
                        )
                    ]) {
                        sh """
                        git clone https://${GIT_USER}:${GIT_TOKEN}@github.com/akritisrivastava22/social-media-app.git temp-repo
                        cd temp-repo
                        git checkout ${branch}
                        """
                    }
                }
            }
        }

        stage('Build Artifact') {
            steps {
                script {
                    // Use lock to ensure one build per branch at a time
                    lock(resource: "branch-${env.BRANCH_NAME}") {
                        sh 'cd temp-repo && mvn clean package -DskipTests'
                    }
                }
            }
        }

        stage('Verify Artifact') {
            steps {
                sh 'ls -lrt temp-repo/target/'
            }
        }

        stage('Upload to JFrog') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'jfrog-creds',
                        usernameVariable: 'JFROG_USER',
                        passwordVariable: 'JFROG_PASS'
                    )
                ]) {
                    sh '''
                    ARTIFACT=$(ls temp-repo/target/*.jar | head -n 1)
                    BRANCH_NAME=$(git -C temp-repo rev-parse --abbrev-ref HEAD)
                    SAFE_BRANCH=$(echo "$BRANCH_NAME" | tr '/' '-')

                    echo "Uploading artifact: $ARTIFACT"

                    curl -u $JFROG_USER:$JFROG_PASS -T $ARTIFACT "$JFROG_URL/${SAFE_BRANCH}-${BUILD_NUMBER}-$(basename $ARTIFACT)"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline SUCCESS ✅ Build completed and artifact uploaded to JFrog'
        }

        failure {
            echo 'Pipeline FAILED ❌ Check logs for details'
        }

        always {
            echo 'Pipeline execution finished (success or failure)'
        }
    }
}
