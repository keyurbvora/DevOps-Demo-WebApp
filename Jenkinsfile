pipeline {
    agent any
    tools { 
        maven 'Maven3.6.3' 
        jdk 'JDK' 
    }
	
    stages {
		stage ('Initialize') {
            steps {
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                ''' 
            }
        }
		
		stage ('static code analysis') {
           steps {
                sh 'mvn validate' 
				withSonarQubeEnv('sonarqube') {
					sh "mvn ${SONAR_MAVEN_GOAL} -Dsonar.host.url=${SONAR_HOST_URL}  -Dsonar.projectKey=WEBPOC:AVNCommunication -Dsonar.sources=. -Dsonar.tests=. -Dsonar.inclusions=**/test/java/servlet/createpage_junit.java -Dsonar.test.exclusions=**/test/java/servlet/createpage_junit.java -Dsonar.login=admin -Dsonar.password=sonar"
				}
            }
		}
    }
}
