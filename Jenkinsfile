echo 'Starting application merge'

def buildFeatureBranch() {
    test()
    build()
    sonar()
    artifactory()
}

def buildDevelopBranch() {
    test()
    build()
    sonar()
    artifactory()
}

def buildReleaseBranch() {
    test()
    build()
    sonar()
    artifactory()
}

def buildMasterBranch() {
    test()
    build()
    sonar()
    artifactory()
}

def buildHotfixBranch() {
    test()
    build()
    sonar()
    artifactory()
}

    node('master') {
        stage('Setup') {
            checkout scm
            def branchName = env.GIT_BRANCH

            if (branchName == 'feature') {
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
                error "Branch ${branchName} is not recognized"
                }
            }
        }

        void test() {
        stage('Test') {
            //If test fails, build will stop here
		    echo 'Testing...'
		    sh './gradlew test'
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

        void sonar() {
        stage('SonarQube Analysis') {
		    echo 'SonarQube...'
		    withSonarQubeEnv('SonarQube') {
		    sh './gradlew sonarqube -Dsonar.projectVersion=0.1'
		        }
            }
        }

        void artifactory() {
        stage('Publish Snapshot to Artifactory') {
		    echo 'Artifactory...'
            sh './gradlew publish'
            }
        }