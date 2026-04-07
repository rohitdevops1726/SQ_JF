def registry= 'https://trialra58da.jfrog.io/'
pipeline {
agent any
environment {
	PATH= "/opt/maven/bin:$PATH"
}

stages {
	
	stage ("Build"){

		steps {
			
			echo "---------Build Started---------"

			sh 'mvn clean deploy -Dmaven.test.skip=true'

			echo "---------Build Completed-------"
		}
	}	

	stage ('Test'){

		steps {

			echo "----------Unit Test Started---------"

			sh 'mvn surefire-report:report'

			echo "----------Unit Test Completed-------"

		}
	}

	stage ('SonarQube Analysis'){

		environment{

			scannerHome= tool 'mysonarqube-tool'
		}

		steps {
			
			withSonarQubeEnv('mysonarqube-server'){
			
				sh "${scannerHome}/bin/sonar-scanner"
			}
		}		
	}	


	stage ('Jar Publish'){

		steps {

			script {

				echo '<--------Jar Publish Started-------->'
				def server= Artifactory.newServer url: registry + "/artifactory", credentialsId: "myjfrog"
				def properties= "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}"
				def uploadSpec= """{
					"files": [
						
						{
							"pattern": "jarstaging/(*)",
							"target": "rohitdevops-libs-release-local/{1}",
							"flat": "false",
							"props": "${properties}",
							"exclusions": ["*.sha1", "*.md5"]
						}
					]
					
				}"""
				def buildInfo= server.upload(uploadSpec)
				buildInfo.env.collect()
				server.publishBuildInfo(buildInfo)
				echo '<----------Jar Publish Ended--------->' 
			}
		}
	}

}

}
