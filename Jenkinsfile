echo 'Starting application merge'

def buildFeatureBranch() {
    test()
    build()
    sonar()
    //merge() Commented out to test Jenkins merging
    artifactorySnapshot()
}

def buildDevelopBranch() {
    test()
    build()
    sonar()
    artifactorySnapshot()
}

def buildReleaseBranch() {
    test()
    build()
    sonar()
    artifactoryRelease()
}

def buildMasterBranch() {
    test()
    build()
    sonar()
    artifactorySnapshot()
}

def buildHotfixBranch() {
    test()
    build()
    sonar()
    artifactorySnapshot()
}

def scmVars

node('master') {
    stage('Setup') {
        scmVars = checkout scm

        if (scmVars.GIT_BRANCH == 'origin/feature') {
            buildFeatureBranch()
        /*} else if (branchName.startsWith('develop')) {
            buildDevelopBranch()
        } else if (branchName.startsWith('release/')) {
            buildReleaseBranch()
        } else if (branchName.startsWith('master')) {
            buildMasterBranch()
        } else if (branchName.startsWith('hotfix/')) {
            buildHotfixBranch()*/
        } else {
            error "Branch ${scmVars.GIT_BRANCH} is not recognized"
        }
    }
}

    void test() {
    stage('Test') {
        //If test fails, build will stop here
        echo 'Testing...'
	    sh './gradlew clean test'
		junit allowEmptyResults: true, testResults: 'build/test-results/test/*.xml'
		publishHTML([allowMissing: true,
		    alwaysLinkToLastBuild: true,
		    keepAll: false,
		    reportDir: 'build/reports/tests/test',
		    reportFiles: 'index.html',
		    reportName: 'Jacoco Exmaple Gradle Test Results',
		    reportTitles: ''
		    ])
        }
    }

    void build() {
    stage('Build') {
	    echo 'Building...'
		sh './gradlew assemble'
        }
    }

    void email() {
    stage('Email') {
		echo 'Emailing...'
        emailext (
        subject: "STARTED: Job '${scmVars.JOB_NAME} [${scmVars.BUILD_NUMBER}]'", 
        body: """<p>STARTED: Job '${scmVars.JOB_NAME} [${scmVars.BUILD_NUMBER}]':</p>
	        <p>*********Released**********</p>
		    <p>Published to Artifactory</p>
            <p>Check console output at "<a href="${scmVars.BUILD_URL}">${scmVars.JOB_NAME} [${scmVars.BUILD_NUMBER}]</a>"</p>""",
        to: "yuseff.perry@capgemini.com"
            )
        }
    }

    void sonar() {
    stage('SonarQube Analysis') {
		echo 'SonarQube...'
		withSonarQubeEnv('SonarQube') {
		sh './gradlew sonarqube -Dsonar.projectVersion=0.1'
		    }
        }
    }

    void merge() {
    stage('Merge to develop') {
	    echo 'Merging...'
        //Compare differences between feature and develop
        sh 'git diff origin/develop'
        //One hour to answer
        timeout(time: 1, unit: 'HOURS') {
            input message: 'Would you like to auto merge?', submitter: 'feature branch 1'
        }
        //Merging to develop branch
        sh 'git checkout origin/develop'
        sh 'git fetch origin feature'
        sh 'git merge origin/feature'
        sh 'git push origin HEAD:develop'
        }
    }

    void artifactorySnapshot() {
    stage('Publish Snapshot to Artifactory') {
        echo 'Artifactory Snapshot...'
        sh './gradlew publish'
        }
    }

    void artifactoryRelease() {
    stage('Publish Release to Artifactory') {
        echo 'Artifactory Release...'
        sh './gradlew publish'
        }
    }