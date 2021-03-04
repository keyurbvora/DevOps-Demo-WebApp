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
				//withSonarQubeEnv('sonarqube') {
					//sh "mvn ${SONAR_MAVEN_GOAL} -Dsonar.host.url=${SONAR_HOST_URL}  -Dsonar.projectKey=WEBPOC:AVNCommunication -Dsonar.sources=. -Dsonar.tests=. -Dsonar.inclusions=**/test/java/servlet/createpage_junit.java -Dsonar.test.exclusions=**/test/java/servlet/createpage_junit.java -Dsonar.login=admin -Dsonar.password=sonar"
				//}
            }
		}

		stage ('Build') {
           steps {
                sh 'mvn compile' 
            }
			//post {
				//always {
					//jiraSendBuildInfo site: 'devopsbc-b3-t3.atlassian.net'
					//jiraSendBuildInfo branch: 'DEVOPS-1', site: 'devopsbc-b3-t3.atlassian.net'
					//jiraSendBuildInfo site: 'devopsbc-b3-t3.atlassian.net', issueKeys: ['DEVOPS*]
      // }
   //}
		}
		
		stage ('Package') {
           steps {
                sh 'mvn package' 
            }
		}
		
		stage ('Publish to artifactory') {
			steps {
				script {
				def server = Artifactory.server('artifactory')
				def rtMaven = Artifactory.newMavenBuild()
				rtMaven.resolver server: server, releaseRepo: 'libs-release', snapshotRepo: 'libs-snapshot'
				rtMaven.deployer server: server, releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot-local'
				rtMaven.deployer.artifactDeploymentPatterns.addInclude("*.war")
				def buildInfo = rtMaven.run pom: 'pom.xml', goals: 'install'
				server.publishBuildInfo buildInfo
				}
			}
		}
		
		stage ('Deploy to QA') {
           steps {
			deploy adapters: [tomcat8(credentialsId: '7c688857-573b-45f8-a4ec-fd7975194aa4', path: '', url: 'http://172.31.1.112:8080')], contextPath: '/QAWebapp', onFailure: false, war: '**/*.war'
			}
		}

		stage ('Run UI Tests') {
           steps {
                sh 'mvn -f functionaltest/pom.xml  test' 
				publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\functionaltest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'UI Test Report', reportTitles: ''])
            }
		}
		//stage ('Run Performance Test') {
		//	steps{
		//		blazeMeterTest credentialsId: 'BlazeMeter', testId: '9014608.taurus', workspaceId: '756649'
		//	}
		//}
		stage ('Deploy to Prod') {
           steps {
			deploy adapters: [tomcat8(credentialsId: '7c688857-573b-45f8-a4ec-fd7975194aa4', path: '', url: 'http://172.31.7.79:8080')], contextPath: '/ProdWebapp', onFailure: false, war: '**/*.war'
			}
		}
		
		stage ('Run Acceptance Tests') {
           steps {
                sh 'mvn -f Acceptancetest/pom.xml  test' 
				publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\Acceptancetest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'Sanity Test Report', reportTitles: ''])
            }
		}	
	}
	post {
    success {
      slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
    }
    failure {
      slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
    }
  }
}
