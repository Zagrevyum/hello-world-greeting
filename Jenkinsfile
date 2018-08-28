node('ubuntu-slave') {
	stage('Poll') {
 		checkout scm
			}
	stage('Build'){
		withMaven(maven: 'M3') {
 		sh 'mvn clean verify -DskipITs=true';
			}
		   }
	stage('Unit Testing'){
 		junit '**/target/surefire-reports/TEST-*.xml'
 		archive 'target/*.jar'
		}

	stage('Static Code Analysis'){
		withMaven(maven: 'M3') {
			withSonarQubeEnv('SonarQube'){
 				sh 'mvn clean verify sonar:sonar -Dsonar.projectName=evaluation-project -Dsonar.projectKey=evaluation-project -Dsonar.projectVersion=$BUILD_NUMBER -Dsonar.language=java';
					}
				}
			}

	stage ('Integration Test'){
		withMaven(maven: 'M3') {
 			sh 'mvn clean verify -Dsurefire.skip=true';
			}
 		junit '**/target/failsafe-reports/TEST-*.xml'
 		archive 'target/*.jar'
		}
	stage ('Publish'){
 		def server = Artifactory.server 'Default Artifactory Server'
		 def uploadSpec = """{
 			"files": [
 				{
 				"pattern": "target/hello-0.0.1.war",
 				"target": "first-project/${BUILD_NUMBER}/",
 				"props": "Integration-Tested=Yes;Performance-Tested=No"
 }
 				]
 					}"""
 		server.upload(uploadSpec)
}
}
