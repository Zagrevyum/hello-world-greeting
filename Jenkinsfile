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
 		def server = Artifactory.server 'Default Artifactory'
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
stash includes:
	'target/hello-0.0.1.war,src/pt/Hello_World_Test_Plan.jmx',
 	 name: 'binary'

 stage ('Start Tomcat'){
 sh '''cd /home/sagrevyum/tomcat/bin
 ./startup.sh''';
 }
	
 stage ('Deploy '){
 unstash 'binary'
 sh 'cp target/hello-0.0.1.war /home/sagrevyum/tomcat/webapps/';
 }
 
	stage ('Performance Testing'){
 sh '''cd /opt/jmeter/bin/
 ./jmeter.sh -n -t $WORKSPACE/src/pt/Hello_World_Test_Plan.jmx -l $WORKSPACE/test_report.jtl''';
 step([$class: 'ArtifactArchiver', artifacts: '**/*.jtl'])
 }
	
 stage ('Promote build in Artifactory'){
 withCredentials([usernameColonPassword(credentialsId:'artifactory', variable: 'credentials')]) {
 sh 'curl -u${credentials} -X PUT "http://192.168.100.49:8081/artifactory/api/storage/first-project/${BUILD_NUMBER}/hello-0.0.1.war?properties=Performance-Tested=Yes"';
 }
 }
}
