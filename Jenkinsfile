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
                sh 'mvn clean package'
            }
        }

        stage('Test') {
            steps {
                echo "Running Maven tests..."
                sh 'mvn test'
            }
        }    

        stage('Deploy') {
            steps {
                echo "Deploying the Artifact..."
                sh 'scp /home/ubuntu/workspace/SonarQube/target/hello-1.0.war ubuntu@54.87.19.68:/home/ubuntu/apache-tomcat-9.0.109/webapps'
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

    } // End of stages

    post {
        success {
            echo "✔️ BUILD AND TEST STAGE SUCCESSFUL...!!!"
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
