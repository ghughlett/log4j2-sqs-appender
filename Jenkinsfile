String branchName='develop'
String mvnConfig='606ddd86-1cb6-42f4-9362-f2108d05a89e'
String gitCred='github-ghughlett'

pipeline {
	environment {
	  MVN_SET = credentials('maven_secret_settings')
	  RUN_STAGE = 'false'
	  SKIP_PREPARE = 'true'
	  CURRENT_VERSION = 'v1.0.7'
	  IS_SNAPSHOT = readMavenPom(file: 'pom.xml').version.contains('SNAPSHOT')
	}
	agent any
    options {
        timeout(time: 30) // minutes; Cloudhub deploy, check if started is an issue
    }
    tools {
        maven 'Maven 3.6.3-Brady'
    }
	stages {
		stage('Build') {
			steps {
			    script {
            		branchName=env.BRANCH_NAME
            		echo branchName
            		echo $IS_SNAPSHOT
			    }
                withMaven(mavenSettingsConfig: mvnConfig) {
       				sh 'mvn clean compile help:effective-settings'
                }
			}
			post {
				success {
					echo '******** Build succeeded'
				}
				failure {
					echo '!!!!!!! Build failed'
					error('Stopping the build')
				}
			}
		}
    	stage('Unit Test') {
      		steps {
                withMaven(mavenSettingsConfig:  mvnConfig) {
                    sh "mvn clean test"
                }
            }
			post {
				success {
					echo '******** Unit Test succeeded'
				}
				failure {
					echo '!!!!!!! Unit Test failed'
					error('Stopping the build')
				}
			}
    	}
    	stage('Release Notes Generation') {
    	    when {
    	        branch 'master'
    	        environment name: 'RUN_STAGE', value: 'true'
    	    }
            steps {
			  withCredentials([usernamePassword(credentialsId: gitCred, passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
			    sh '''
                    touch changelog.txt
                    git log --pretty="%h - %s%b (%an)" $(git tag | tail -n1)...HEAD > changelog.txt
                    git add changelog.txt
	             '''
              }
      		}
			post {
				success {
				    echo '******** Release notes created'
	                sh 'cat changelog.txt'
				}
				failure {
					echo '!!!!!!! Release notes generation failed'
					error('Stopping the build')
				}
			}
    	}
        stage('Release-Merge Master') {
    	    when {
    	        branch 'master'
    	        environment name: 'SKIP_PREPARE', value: 'true'
    	        environment name: 'RUN_STAGE', value: 'true'
    	    }
            steps {
                script {
                    pom = readMavenPom(file: 'pom.xml')
                    projectArtifactId = pom.getArtifactId()
                    projectGroupId = pom.getGroupId()
                    projectVersion = pom.getVersion()
                    projectName = pom.getName()
                }
                echo "Merging ${projectArtifactId}:${projectGroupId}:{projectVersion}"

                withCredentials([usernamePassword(credentialsId: gitCred, passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                    withMaven(mavenSettingsConfig:  mvnConfig) {
                        sh '''
                            mvn scm:validate
                        '''
                        sh '''
                            mvn build-helper:parse-version versions:set -DnewVersion=\'${parsedVersion.majorVersion}.${parsedVersion.minorVersion}.${parsedVersion.nextIncrementalVersion}-SNAPSHOT\'
                        '''
                        sh '''
                            mvn scm:checkin -Dmessage="checkin" scm:tag -Dtag=$CURRENT_VERSION
                        '''
                    }
                }
            }
			post {
				success {
					echo '******** Merge Master succeeded'
				}
				failure {
					echo '!!!!!!! Merge Master failed'
					error('Stopping the build')
				}
			}
		}
        stage('Release-GitHub Artifact Deployment') {
    	    when {
    	        branch 'master'
    	        environment name: 'SKIP_PREPARE', value: 'true'
    	        environment name: 'RUN_STAGE', value: 'true'
    	    }
            steps {
                withMaven(mavenSettingsConfig:  mvnConfig) {
         	        sh 'mvn clean deploy -s $MVN_SET'
                }
            }
			post {
				success {
					echo '******** GitHub Deployment succeeded'
				}
				failure {
					echo '!!!!!!! GitHub Deployment failed'
					error('Stopping the build')
				}
			}
		}
	}
}