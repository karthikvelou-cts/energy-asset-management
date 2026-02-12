pipeline {
    agent any

    environment {
        CI = 'true'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/cts-karthikvelou/energy-asset-management/'
            }
        }

        stage('Show package info') {
            steps {
                sh '''
                    node -v && npm -v
                    if [ ! -f package.json ]; then
                      echo "package.json not found!"
                      exit 1
                    fi
                    echo "package.json:"
                    cat package.json
                    echo "Available npm scripts:"
                    npm run || true
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm ci || npm install'
            }
        }

        stage('Build') {
            when {
                expression {
                    def pkg = readJSON file: 'package.json'
                    return pkg?.scripts?.build != null
                }
            }
            steps {
                sh 'npm run build'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'dist/**', allowEmptyArchive: true
                }
            }
        }

        stage('Test with Coverage') {
            when {
                expression {
                    def pkg = readJSON file: 'package.json'
                    return pkg?.scripts?.test != null
                }
            }
            steps {
                sh 'npm test'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'coverage/**', allowEmptyArchive: true
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('MySonarQubeServer') {
                        def scannerHome = tool 'SonarScanner'  // Keep this ONLY if configured in Jenkins
                        withCredentials([string(credentialsId: 'SONARQUBE_TOKEN', variable: 'SQ_TOKEN')]) {
                            sh """
                                "${scannerHome}/bin/sonar-scanner" \
                                  -Dsonar.projectKey=energy-asset-management \
                                  -Dsonar.sources=. \
                                  -Dsonar.host.url="${env.SONAR_HOST_URL}" \
                                  -Dsonar.login="${SQ_TOKEN}" \
                                  -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info
                            """
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            when { branch 'main' }
            steps {
                echo 'Deploying build artifacts...'
            }
        }
    }

    post {
        success { echo 'Pipeline completed successfully!' }
        failure { echo 'Pipeline failed. Please check logs.' }
        always {
            archiveArtifacts artifacts: 'dist/**', allowEmptyArchive: true
        }
    }
}
