pipeline {
    agent any

    tools {
        maven "Maven"
        jdk "Java-21"
    }

    stages {
        stage('Clean Workspace') {
            steps {
                echo "Cleaning Workspace......"
                deleteDir()
            }
        }

        stage('Checkout') {
            steps {
                git branch: "main", url: 'https://github.com/bharathsavadatti447/SonarQube.git'
            }
        }

        stage('Build') {
            steps {
                echo "Compiling Maven project..."
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                echo "Running Maven tests..."
                sh 'mvn test'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                echo "Deploying the WAR to Tomcat server..."
                }
        }

        stage('SonarQube Analysis') {
            steps {
                echo "Running SonarQube Analysis..."
                withSonarQubeEnv('Sonar') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        // 🔹 New Stages for Artifactory Integration
        stage('Prepare Artifact for Artifactory') {
            steps {
                echo "Preparing artifact for upload..."
                sh 'mkdir -p build/output && cp target/*.war build/output/'
            }
        }

        stage('Push WAR to Artifactory') {
            steps {
                script {
                    echo "Uploading WAR file to Artifactory..."
                    def server = Artifactory.server('JFrog')
                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "build/output/*.war",
                                "target": "generic-local/java-builds/${env.JOB_NAME}/${env.BUILD_NUMBER}/"
                            }
                        ]
                    }"""
                    server.upload(uploadSpec)
                }
            }
        }
    }

    post {
        success {
            echo "✔️ BUILD, TEST & UPLOAD SUCCESSFUL...!!!"
            junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
            archiveArtifacts artifacts: 'target/*.war', onlyIfSuccessful: true
        }

        failure {
            echo "❌ Failure..!!!"
            mail to: 'bharath.savadatti447@gmail.com',
                 subject: "Failed: ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                 body: "Job '${env.JOB_NAME}' (${env.BUILD_URL}) failed"
        }

        always {
            echo "🔸🔸🔸You executed the build🔸🔸🔸!!!"
        }
    }
}
