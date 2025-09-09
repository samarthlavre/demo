pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                script {
                    if (fileExists('mvnw.cmd')) {
                        bat 'mvnw.cmd -B -DskipTests clean package'
                    } else {
                        bat 'mvn -B -DskipTests clean package'
                    }
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    if (fileExists('mvnw.cmd')) {
                        bat 'mvnw.cmd -B test'
                    } else {
                        bat 'mvn -B test'
                    }
                }
            }
            post {
                always {
                    script {
                        if (fileExists('target/surefire-reports')) {
                            junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'
                        } else {
                            echo "⚠️ No test reports found, skipping JUnit publish."
                        }
                    }
                }
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Run Application') {
            steps {
                script {
                    // Use %%i for bat loops inside Jenkins
                    def jarFile = bat(
                        script: 'for %%i in (target\\*.jar) do @echo %%~nxi',
                        returnStdout: true
                    ).trim()

                    echo "Starting ${jarFile}"

                    bat """
                        start /B java -jar target\\${jarFile} > app.log 2>&1
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Build and Run Successful!'
        }
        failure {
            echo '❌ Build Failed.'
        }
    }
}
