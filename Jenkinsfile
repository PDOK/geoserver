#!groovy

properties([
	buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '10', numToKeepStr: '5')), 
	[$class: 'ScannerJobProperty', doNotScan: false], 
	[$class: 'GithubProjectProperty', projectUrlStr: 'http://github.so.kadaster.nl/PDOK/geoserver/'], 
	[$class: 'CopyArtifactPermissionProperty', projectNames: '*'])
])

gitCommit = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
gitCommitShort = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()

GEOSERVER_VERSION = "2.10.1"
RELEASE_VERSION = "${GEOSERVER_VERSION}-PDOK-" + (env.BUILD_NUMBER ? env.BUILD_NUMBER : 'handbuilt') + "-${gitCommitShort}"

node() {
	deleteDir()
	
	def mvnHome = tool 'Maven 3.3.9'
	def repoBranch = env.BRANCH_NAME
	env.PATH = "${mvnHome}/bin:${env.PATH}"
	
	sh 'git config --global user.email "pdok.koala@kadaster.nl"'
	sh 'git config --global user.name "PDOK_Koala"'
	
	try {
		stage("Checkout from GIT") {
			checkoutGit{
				branch = repoBranch
			}
		}
		stage("Set geoserver version") {
			dir("src") {
				sh '''#!/bin/bash -l
					function get_pom_version() {
					  local pom=$1
					  echo `grep "<version>" $pom | head -n 1 | sed 's/<.*>\(.*\)<\/.*>/\1/g' | sed 's/ //g'`
					}
					new_ver=$RELEASE_VERSION
					old_ver=`get_pom_version src/pom.xml`
					echo "updating version numbers from $old_ver to $new_ver"
					find src -name pom.xml -exec sed -i "s/$old_ver/$new_ver/g" {} \;
					find doc -name conf.py -exec sed -i "s/$old_ver/$new_ver/g" {} \;
				'''
			}
		}
		stage("Build geoserver") {
			dir("src") {
				sh "mvn -B release:prepare release:perform -DskipTests=true"
			}
		}
	} catch (exc) {
		currentBuild.result = "Failure"
		echo "Caught: ${exc}"
	}
}


