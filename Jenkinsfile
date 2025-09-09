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
                    echo "Starting demo-1.0-SNAPSHOT.jar"
                    bat """
                        start /B java -jar target/demo-1.0-SNAPSHOT.jar > app.log 2>&1
                    """
                }
            }
        }

        stage('Verify Application') {
            steps {
                script {
                    echo "Checking application log..."
                    bat "type app.log"
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
