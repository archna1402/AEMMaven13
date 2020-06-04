pipeline {
    agent any
    tools {
        jdk 'jdk-8'
        maven 'maven-3.6.3'
    }
    
    stages {

        stage('Git Checkout') {
            steps {
                echo 'Checking out git repository'
		git 'https://github.com/adobe/aem-guides-wknd.git'
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
                        error "ERROR! Stop pipeline excution!"
                    }
                }
            }
        }

		stage('Upload Artifacts to Nexus') {
		    
				steps{
					script{
						def mavenPom = readMavenPom file: 'pom.xml'
						nexusArtifactUploader artifacts: 
							[[artifactId: 'aem-guides-wknd',
							    classifier: 'apps', 
								file: "/var/lib/jenkins/.m2/repository/com/adobe/aem/guides/aem-guides-wknd.ui.apps/${mavenPom.version}/aem-guides-wknd.ui.apps-${mavenPom.version}.zip",  							
								type: 'zip'], 
								[artifactId: 'aem-guides-wknd',
								classifier: 'content',
								file: " /var/lib/jenkins/.m2/repository/com/adobe/aem/guides/aem-guides-wknd.ui.content/${mavenPom.version}/aem-guides-wknd.ui.content-${mavenPom.version}.zip",
								type: 'zip']],
								credentialsId: 'nexus3', 
								groupId: 'com.adobe.aem.guides', 
								nexusUrl: '3.6.165.101:8081', 
								nexusVersion: 'nexus3', 
								protocol: 'http', 
								repository: 'aem-wknd-snapshots', 
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
        success {
            echo 'Clear dispatcher cache'
            echo 'Clear CDN cache'
            echo "Build Success : : ${env.BUILD_NUMBER}"
			echo "RESULT: ${currentBuild.currentResult}"
        }
        failure {
            echo "Build Failure : ${env.BUILD_NUMBER}"
			echo "RESULT: ${currentBuild.currentResult}"
        }
		always {
            mail to: 'archna@epsilon.com',
            subject: "Status of Build:${currentBuild.fullDisplayName}, ${env.JOB_NAME} - ${currentBuild.result}",
            body: "Job ${currentBuild.result} - ${env.JOB_NAME} build: ${env.BUILD_NUMBER} has result ${currentBuild.result}"
        }
    }
}
