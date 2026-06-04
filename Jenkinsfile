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
                // This will checkout the branch that Jenkins detects
                checkout scm
            }
        }

        stage('Build Artifact') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Verify Artifact') {
            steps {
                sh 'ls -lrt target/'
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
                    ARTIFACT=$(ls target/*.jar | head -n 1)
                    BRANCH_NAME=$(git rev-parse --abbrev-ref HEAD)
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
