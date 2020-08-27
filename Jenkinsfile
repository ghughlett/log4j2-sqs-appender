String branchName='develop'

pipeline {
	environment {
	  MVN_SET = credentials('maven_secret_settings')
	  SKIP_PREPARE = 'true'
	  CURRENT_VERSION='v1.0.6'
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
			    }
                withMaven(mavenSettingsConfig: '606ddd86-1cb6-42f4-9362-f2108d05a89e') {
       				sh 'mvn clean compile -s $MVN_SET help:effective-settings'
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
                withMaven(mavenSettingsConfig: '606ddd86-1cb6-42f4-9362-f2108d05a89e') {
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
    	    }
            steps {
			  withCredentials([usernamePassword(credentialsId: 'github-ghughlett', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
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
		            //sh 'cat CHANGELOG-music-factory.md'
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

			    //withCredentials([usernamePassword(credentialsId: 'github-ghughlett', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
			    //    sh '''
			    //        git commit -am "release ${projectArtifactId}:${projectVersion} updated"
                //        //git remote set-url origin https://github.com/ghughlett/log4j2-sqs-appender
                //        git remote set-url origin https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/ghughlett/log4j2-sqs-appender
                //        git tag -f $CURRENT_VERSION
                //        git push origin master --follow-tags
                //    '''
                //}

                withCredentials([usernamePassword(credentialsId: 'github-ghughlett', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                    withMaven(mavenSettingsConfig: '606ddd86-1cb6-42f4-9362-f2108d05a89e') {
                        sh '''
                            mvn scm:validate
                            mvn build-helper:parse-version versions:set -DnewVersion=${parsedVersion.majorVersion}.${parsedVersion.minorVersion}.${parsedVersion.nextIncrementalVersion}-SNAPSHOT
                            git tag -d $CURRENT_VERSION
                            mvn scm:checkin -Dmessage="checkin" scm:tag -Dtag="$CURRENT_VERSION"
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
        stage('Release-Artifact Deployment') {
    	    when {
    	        branch 'master'
    	        environment name: 'SKIP_PREPARE', value: 'true'
    	    }
            steps {
                withMaven(mavenSettingsConfig: '606ddd86-1cb6-42f4-9362-f2108d05a89e') {
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