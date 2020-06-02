pipeline {
    agent any
    tools {
        jdk 'jdk-8'
        maven 'maven-3.6.3'
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

		stage('Upload Artifacts to Nexus') {
		    
				steps{
					script{
						def mavenPom = readMavenPom file: 'pom.xml'
						nexusArtifactUploader artifacts: 
							[[artifactId: 'AEMMaven13',
							    classifier: 'apps', 
								file: "/var/lib/jenkins/.m2/repository/AEMMaven13/AEMMaven13.ui.apps/${mavenPom.version}/AEMMaven13.ui.apps-${mavenPom.version}.zip", 
								type: 'zip'], 
								[artifactId: 'AEMMaven13',
								classifier: 'content',
								file: "/var/lib/jenkins/.m2/repository/AEMMaven13/AEMMaven13.ui.content/${mavenPom.version}/AEMMaven13.ui.content-${mavenPom.version}.zip", 
								type: 'zip']],
								credentialsId: 'nexus3', 
								groupId: 'AEMMaven13', 
								nexusUrl: '3.6.165.101:8081', 
								nexusVersion: 'nexus3', 
								protocol: 'http', 
								repository: 'aem-nexus-repo', 
								version: "${mavenPom.version}"
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
            mail to: 'archna@epsilon.com',
            subject: "Status of Build:${currentBuild.fullDisplayName}, ${env.JOB_NAME} - ${currentBuild.result}",
            body: "Job ${currentBuild.result} - ${env.JOB_NAME} build: ${env.BUILD_NUMBER} has result ${currentBuild.result}"
        }
        failure {
            echo 'Send Failure Email'
        }
    }
}
