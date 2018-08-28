Node("ubuntu-slave){
	stage('Poll') {
 		scm checkout
			}
	stage('Build'){
 		sh 'mvn clean verify -DskipITs=true';
			}
	stage('Static Code Analysis'){
 		sh 'mvn clean verify sonar:sonar -Dsonar.projectName=evaluation-project -Dsonar.projectKey=evaluation-project -Dsonar.projectVersion=$BUILD_NUMBER';
		}
	stage('Unit Testing'){
 		junit '**/target/surefire-reports/TEST-*.xml'
 		archive 'target/*.jar'
		}
	stage ('Integration Test'){
 		sh 'mvn clean verify -Dsurefire.skip=true';
 		junit '**/target/failsafe-reports/TEST-*.xml'
 		archive 'target/*.jar'
		}
	stage ('Publish'){
 		def server = Artifactory.server 'Default Artifactory Server'
		 def uploadSpec = """{
 			"files": [
 				{
 				"pattern": "target/hello-0.0.1.war",
 				"target": "helloworld-greeting-project/${BUILD_NUMBER}/",
 				"props": "Integration-Tested=Yes;Performance-Tested=No"
 }
 				]
 					}"""
 		server.upload(uploadSpec)
}
}
