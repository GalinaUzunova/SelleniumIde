pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm  // Pulls code from SCM (Git, etc.)
            }
        }

        stage('Install .NET SDK') {
            steps {
                script {
                    // Ensure correct .NET version is installed
                    bat "dotnet --list-sdks"
                    bat "dotnet --version"
                }
            }
        }

        stage('Restore NuGet Packages') {
            steps {
                bat "dotnet restore \"${SOLUTION_NAME}\""
            }
        }

        stage('Build') {
            steps {
                bat """
                    dotnet build \"${SOLUTION_NAME}\" \
                    --configuration ${CONFIGURATION} \
                    --no-restore \
                    /p:Version=1.0.${env.BUILD_NUMBER}
                """
            }
        }

        stage('Run Tests') {
            steps {
                bat """
                    dotnet test \"${SOLUTION_NAME}\" \
                    --configuration ${CONFIGURATION} \
                    --no-build \
                    --logger \"trx;LogFileName=testresults.trx\" \
                    --results-directory \"${ARTIFACTS_DIR}\"
                """
            }
            post {
                always {
                    // Publish test results to Jenkins
                    junit allowEmptyResults: true, testResults: "${ARTIFACTS_DIR}\\*.trx"
                }
            }
        }

        stage('Publish Artifacts (Optional)') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                bat "dotnet publish \"${SOLUTION_NAME}\" --configuration ${CONFIGURATION} --output \"${ARTIFACTS_DIR}\\publish\""
                archiveArtifacts artifacts: "${ARTIFACTS_DIR}\\**\\*.*", fingerprint: true
            }
        }
    }

    post {
        always {
            script {
                echo "Cleaning up workspace..."
                cleanWs()  // Optional: Clean workspace after build
            }
        }
        success {
            echo "Build succeeded! 🎉"
        }
        failure {
            echo "Build failed! ❌"
        }
    }
}