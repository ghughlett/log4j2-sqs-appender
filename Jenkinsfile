String branchName='develop'

pipeline {
	environment {
	  MVN_SET = credentials('maven_secret_settings')
	  SKIP_PREPARE = 'true'
      script {
          pom = readMavenPom(file: 'pom.xml')
          projectArtifactId = pom.getArtifactId()
          projectGroupId = pom.getGroupId()
          projectVersion = pom.getVersion()
          CURRENT_VERSION=pom.getVersion()
          projectName = pom.getName()
      }
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
                withMaven(mavenSettingsConfig: '71d7c536-d52e-4ade-9b4e-7cc7a196a327') {
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
                withMaven(mavenSettingsConfig: '71d7c536-d52e-4ade-9b4e-7cc7a196a327') {
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

			    withCredentials([usernamePassword(credentialsId: 'github-ghughlett', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
			        sh '''
			            git commit -am "release ${projectArtifactId}:${projectVersion} updated"
                        git remote set-url origin https://github.com/ghughlett/log4j2-sqs-appender
                        git tag -f $CURRENT_VERSION
                        git push origin master --follow-tags
                    '''
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
                withMaven(mavenSettingsConfig: '71d7c536-d52e-4ade-9b4e-7cc7a196a327') {
         	        sh 'mvn clean deploy'
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
        stage('Prepare Master for Next Release') {
    	    when {
    	        branch 'master'
    	        environment name: 'SKIP_PREPARE', value: 'false'
    	    }
            steps {
                script {
                    pom = readMavenPom(file: 'pom.xml')
                    projectArtifactId = pom.getArtifactId()
                    projectGroupId = pom.getGroupId()
                    projectVersion = pom.getVersion()
                    projectName = pom.getName()
                }
                echo "Updating ${projectArtifactId}:${projectGroupId}:{projectVersion}-SNAPSHOT"

                withMaven(mavenSettingsConfig: '71d7c536-d52e-4ade-9b4e-7cc7a196a327') {
         	        sh 'mvn build-helper:parse-version versions:set -DnewVersion=\${parsedVersion.majorVersion}.\${parsedVersion.minorVersion}.\${parsedVersion.nextIncrementalVersion}-SNAPSHOT'
                }

			    withCredentials([usernamePassword(credentialsId: 'github-ghughlett', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
			        sh '''
			            git add .
			            git commit -am "release ${projectArtifactId}:${projectVersion} updated"
                        git remote set-url origin https://github.com/ghughlett/log4j2-sqs-appender
                        git tag af v${projectVersion}
                        git push origin master --follow-tags
                    '''
                }
            }
			post {
				success {
					echo '******** Prepare Master for Next Release succeeded'
				}
				failure {
					echo '!!!!!!! Prepare Master for Next Release failed'
					error('Stopping the build')
				}
			}
		}
	}
}