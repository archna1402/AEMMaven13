pipeline {
    agent any
    tools {
        jdk 'jdk-8'
        maven 'maven-3.6.3'
    }
    environment {
        PATH = "/usr/local/apache-maven-3.6.3:$PATH"
    }
    stages {

        stage('Git Checkout') {
            steps {
                echo 'This stage will be executed first'
                git 'https://github.com/archna1402/AEMMaven13.git'
            }
        }

        stage('Maven Build') {
            steps {
                //input ('Do you want to proceed?')
                script {
                    try {
                        sh "mvn clean install"
                        echo "RESULT: ${currentBuild.currentResult}"
                    } catch (Throwable e) {
                        echo "RESULT: ${currentBuild.currentResult}"
                        echo "Send Error Email!"
                        error "Stops the pipeline excution!"
                    }
                }
            }
        }

        stage('Archive Artifacts') {
            steps {
                dir('/var/lib/jenkins/.m2/repository/AEMMaven13/AEMMaven13.core/1.0-SNAPSHOT/') {
                    archiveArtifacts artifacts: '*.jar*',
                    onlyIfSuccessful: true
                }
                dir('/var/lib/jenkins/.m2/repository/AEMMaven13/AEMMaven13.ui.content/1.0-SNAPSHOT/') {
                    archiveArtifacts artifacts: '*.zip*',
                    onlyIfSuccessful: true
                }
            }

        }

        stage('Deploy to AEM server') {

            steps {
                echo "Deply to AEM server"
            }

        }

    }

    post {
        always {
            echo 'cleanup tasks'
            deleteDir() /* clean up our workspace */
        }
        success {
            echo 'Clear dispatcher cache'
            echo 'Clear CDN cache'
            echo 'Send Success Email'
            mail to: 'archna.014@gmail.com',
            subject: "Status of Build:${currentBuild.fullDisplayName}, ${env.JOB_NAME} - ${currentBuild.result}",
            body: "Job ${currentBuild.result} - ${env.JOB_NAME} build: ${env.BUILD_NUMBER} has result ${currentBuild.result}"
        }
        failure {
            echo 'Send Failure Email'
        }
    }
}
