gitCredential = 'adesjardin-bitbucket'
String branchName='develop'
String repository

pipeline {
	environment {
	  MVN_SET = credentials('maven_secret_settings')
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
					if (env.BRANCH_NAME == 'master') {
	            		echo 'Found the MASTER branch'
	        		}
	        		else {
	            		//echo env.BRANCH_NAME
	            		branchName=env.BRANCH_NAME
	            		echo branchName
	            		repository = "git@" + env.GIT_URL.replaceFirst(".+://", "").replaceFirst("/", ":")
	        		}
			    }
   				sh 'mvn clean compile -s $MVN_SET help:effective-settings'
			}
		}
    	stage('Unit Test') {
      		steps {
              sh 'mvn clean test -s $MVN_SET'
              //echo 'skipping this step'
      		}
    	}
        stage('Project Specifics') {
            steps {
                sh '''
                    mvn build-helper:parse-version
                '''
                echo "majorVersion: ${parsedVersion.majorVersion}"
                echo "minorVersion: ${parsedVersion.minorVersion}"
                echo "incrementalVersion: ${parsedVersion.incrementalVersion}"
                echo "qualifier: ${parsedVersion.qualifier}"
                echo "nextMajorVersion: ${parsedVersion.nextMajorVersion}"
                echo "nextMinorVersion: ${parsedVersion.nextMinorVersion}"
                echo "nextIncrementalVersion: ${parsedVersion.nextIncrementalVersion}"

            }
        }
    	stage('Release Notes Generation') {
    	    when {
    	        branch 'master'
    	    }
            steps {
			  withCredentials([usernamePassword(credentialsId: 'github-ghughlett', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
			    sh '''
                    git log --pretty="%h - %s%b (%an)" $(git tag | tail -n1)...HEAD > changelog.txt
                    git add changelog.txt
                    git commit -m "added release notes"
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
        stage('Prepare Release') {
    	    when {
    	        branch 'master'
    	    }
            steps {
                script {
                    pom = readMavenPom(file: 'pom.xml')
                    projectArtifactId = pom.getArtifactId()
                    projectGroupId = pom.getGroupId()
                    projectVersion = pom.getVersion()
                    projectName = pom.getName()
                }
                echo "Building ${projectArtifactId}:${projectVersion}"

                sh '''
                    mvn build-helper:parse-version -DpropertyPrefix=parsedVersion -X
                '''
                //echo "majorVersion: ${parsedVersion.majorVersion}"
                //echo "minorVersion: ${parsedVersion.minorVersion}"
                //echo "incrementalVersion: ${parsedVersion.incrementalVersion}"
                //echo "qualifier: ${parsedVersion.qualifier}"
                //echo "nextMajorVersion: ${parsedVersion.nextMajorVersion}"
                //echo "nextMinorVersion: ${parsedVersion.nextMinorVersion}"
                //echo "nextIncrementalVersion: ${parsedVersion.nextIncrementalVersion}"

                sh '''
                    mvn build-helper:parse-version versions:set -DnewVersion=${parsedVersion.majorVersion}.\${parsedVersion.minorVersion}.\${parsedVersion.nextIncrementalVersion}
                '''


			    withCredentials([usernamePassword(credentialsId: 'github-ghughlett', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                   //mvn --batch-mode -s $MVN_SET release:prepare release:perform -Dusername=${GIT_USERNAME} -Dpassword=${GIT_PASSWORD}
			        sh '''
			            git add .
			            git commit -am "release ${projectArtifactId}:${projectVersion} updated"
                        git remote set-url origin https://github.com/ghughlett/log4j2-sqs-appender
                        git tag af v1.0.1
                        git push origin develop --tags
                    '''
                }
            }
			post {
				success {
					echo '******** Prepare Release succeeded'
				}
				failure {
					echo '!!!!!!! Prepare Release failed'
					error('Stopping the build')
				}
			}
		}
        stage('GitHub Deployment') {
    	    when {
    	        branch 'master'
    	    }
            steps {
     	      sh 'mvn clean deploy -s $MVN_SET'
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