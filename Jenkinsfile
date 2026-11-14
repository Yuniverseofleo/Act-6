
pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    tools {
        nodejs 'NodeJS-24'
    }

    stages {
        stage('Checkout') {
            steps {
                deleteDir()
                checkout scm
            }
        }

        stage('Tests and Coverage') {
            steps {
                sh 'node --version'
                sh 'npm ci'
                sh 'npm run coverage'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'

                    withSonarQubeEnv('SonarQube-Lab') {
                        withCredentials([
                            string(
                                credentialsId: 'sonar-token',
                                variable: 'SONAR_AUTH_TOKEN'
                            )
                        ]) {
                            sh """
                                "${scannerHome}/bin/sonar-scanner" \
                                -Dsonar.host.url="${SONAR_HOST_URL}" \
                                -Dsonar.token="${SONAR_AUTH_TOKEN}"
                            """
                        }
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    script {
                        def gate = waitForQualityGate()

                        if (gate.status != 'OK') {
                            error "Quality gate failed: ${gate.status}"
                        }
                    }
                }
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Archive Artifact') {
            steps {
                sh 'tar -czf "react-app-${BUILD_NUMBER}.tgz" dist Dockerfile'

                archiveArtifacts(
                    artifacts: "react-app-${BUILD_NUMBER}.tgz",
                    fingerprint: true
                )
            }
        }
    }

    post {
        success {
            echo 'Quality gate passed and artifact archived.'
        }

        failure {
            echo 'Inspect the failed stage and SonarQube dashboard.'
        }
    }
}
