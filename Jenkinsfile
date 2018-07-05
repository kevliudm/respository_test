node {
	try {
		def jdk
		def mvnHome
		
		properties([
		    buildDiscarder(logRotator(artifactDaysToKeepStr: '20', artifactNumToKeepStr: '50', daysToKeepStr: '20', numToKeepStr: '50')),
		    pipelineTriggers([pollSCM('')])
	    ])

		stage('Preparation') {
			
			jdk = tool name: 'jdk8'
			env.JAVA_HOME = "${jdk}"
			mvnHome = tool 'maven3'
            echo "jdk installation path is: ${jdk}"
            sh "${jdk}/bin/java -version"
            sh '$JAVA_HOME/bin/java -version'
			
			checkout scm
		}

		stage('Static Analysis') {
			// TODO: Run checkstyle/PMD/findbugs/emma

		}

		stage('Package') { // Run package
			sh "'${mvnHome}/bin/mvn' -U -Dmaven.test.failure.ignore clean package" }

		stage('Archive') {
			archive 'target/*.jar'
			fingerprint 'target/*.jar'
		}

	} catch (e) {
		// If there was an exception thrown, the build failed
		currentBuild.result = "FAILED"
		throw e
	} finally {
		// Success or failure, always send notifications
		notifyBuild(currentBuild.result)
	}
}


// Send email by different build status
def notifyBuild(String buildStatus = 'STARTED') {
	// build status of null means successful
	buildStatus = buildStatus ?: 'SUCCESS'

	// Default values
	def colorName = 'RED'
	def colorCode = '#FF0000'
	def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
	def summary = "${subject} (${env.BUILD_URL})"
	def details = """<p>STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
	  <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>"""

	// Override default values based on build status
	if (buildStatus == 'STARTED') {
		color = 'YELLOW'
		colorCode = '#FFFF00'
	} else if (buildStatus == 'SUCCESS') {
		color = 'GREEN'
		colorCode = '#00FF00'
	} else {
		color = 'RED'
		colorCode = '#FF0000'
	}

	// Send notifications
	emailext body: '$DEFAULT_CONTENT',
	recipientProviders: [
		[$class: 'CulpritsRecipientProvider'],
		[$class: 'DevelopersRecipientProvider'],
		[$class: 'RequesterRecipientProvider']
	],
	replyTo: '$DEFAULT_REPLYTO',
	subject: '$DEFAULT_SUBJECT',
	to: '$DEFAULT_RECIPIENTS'
}