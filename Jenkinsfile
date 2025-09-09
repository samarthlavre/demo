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
                    // If project has mvnw.cmd wrapper, use it; otherwise use installed Maven
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
                    // Publish JUnit test reports
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Archive Artifact') {
            steps {
                // Save the built JAR in Jenkins
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Run Application') {
            steps {
                script {
                    // Find the generated JAR file
                    def jarFile = bat(
                        script: 'for %i in (target\\*.jar) do @echo %i',
                        returnStdout: true
                    ).trim()

                    echo "Starting ${jarFile}"

                    // Run it in background (logs saved to app.log)
                    bat """
                        start /B java -jar ${jarFile} > app.log 2>&1
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
