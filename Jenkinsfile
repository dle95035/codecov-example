
def isOnlyVersionBump() {

    // full path file name.
    def fullFileName
	
	if (null != currentBuild.changeSets) {
		def changeSets = currentBuild.changeSets
	} else {
		return false
	}
    if (changeSets[0].items) {
		def items = changeSets[0].items
	} else {
		return false
	}
	if (items[0].affectedFiles) {
		def affectedFiles = items[0].affectedFiles
	} else {
		return false
	}

    // single changeSet, 2 items , single affectedFile
    if ( 1 == changeSets.size() ) {
        if ( 1 == affectedFiles.size() ) {
            fullFileName = affectedFiles.first().path
            if ( "version.sbt" == fullFileName.substring(fullFileName.lastIndexOf("/")+1) ) {
                return true
            }
        }
    }
    return false
}

///
def _isOnlyVersionBump() {
    def fullFileName
	
    if ( 1 == currentBuild.changeSets.size() ) { 
		if ( 2 >= currentBuild.changeSets[0].items.size() ) { 
			if ( 1 == currentBuild.changeSets[0].items[0].affectedFiles.size() ) { 
				fullFileName = currentBuild.changeSets[0].items[0].affectedFiles.first().path
				if ( "version.sbt" == fullFileName.substring(fullFileName.lastIndexOf("/")+1) ) {
					return true
				}
			}
		}
	}
	return false
}

def cause() {
    def causes = currentBuild.getBuildCauses()
    for (cause in causes) {
        println cause.toString()
        println cause["shortDescription"]
        println cause["_class"]
    }
}

def _cause() {
    def causes = currentBuild.rawBuild.getCauses()
    for (cause in causes) {
        println cause.toString()
        //println cause["shortDescription"]
    }
}


def user() {
	def specificCause = currentBuild.rawBuild.getCause(hudson.model.Cause$UserIdCause)
    println specificCause
}

def merge() {
	def specificCause = currentBuild.rawBuild.getCause(hudson.model.SCMTrigger$SCMTriggerCause)
    println specificCause
}

def isGitHubPush() {
    def causes = currentBuild.rawBuild.getCauses()
    for (cause in causes) {
        if (cause.toString().contains("GitHubPushCause")) {
			return true
		}
    }
	return false
}

def readProp(fileName, key) {
	for (String line : readFile(fileName).split("\r?\n")) {
		if (line.contains(key)) {
			return line.split('=')[1]
		}
	}
	return ""
}

def sendMails() {
    if (SKIP_ALL == 'true') { return }
	receipients = readProp("statics.properties", "receipients").split()
	for (receipient in receipients) {
		println "${receipient}@exabeam.com"
	}	
}

//def target_url = readProp("statics.properties", "target").split()[0]
//def target_pwd = readProp("statics.properties", "target").split()[1]

// every 5 minutes
// H/5 * * * *
pipeline {
    agent any
	environment {
		SKIP_ALL = isGitHubPush()
	}
    stages {
	    stage('Read') {
			steps {
				sh 'ls -la'
				script {
					println readProp("statics.properties", "target").split()[0]
					//println target_pwd
					sendMails()
				}
				
			}
		}
		stage ('Check') {
			when {
				expression {SKIP_ALL == 'false'}
			}
			steps {	
				script {
					sh 'echo hello'
					sh 'ls -la'
					sh 'sbt clean coverage test coverageReport'
					sh 'CODECOV_TOKEN="2b1253d5-3df4-4186-a8b6-9f5eb7db282d"   bash <(curl -s https://codecov.io/bash) '
				}
			}
		}
        stage('Clone repository and build tests') {
			when {
				expression {SKIP_ALL == 'false'}
			}
            steps {
                 sh 'ls -la'
				 _cause()
			} 
         }
     }
	 // https://jenkins.io/doc/pipeline/tour/post/
	 post {
		
		always {
			script {
				if (SKIP_ALL == 'true') {
					echo 'do somthing here'
					return
				}
			}
			echo 'this is it'
		}
		
	 }
}
