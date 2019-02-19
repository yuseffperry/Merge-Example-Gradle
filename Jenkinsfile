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
            def name = env.BRANCH_NAME

            if (name.startsWith('feature/')) {
                buildFeatureBranch()
            } else if (name == 'develop') {
                buildDevelopBranch()
            } else if (name.startsWith('release/')) {
                buildReleaseBranch()
            } else if (name == 'master') {
                buildMasterBranch()
            } else if (name.startsWith('hotfix/')) {
                buildHotfixBranch()
            } else {
                error "Branch ${name} is not recognized"
                }
            }
        }

        def test() {
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

        def build() {
        stage('Build') {
		    echo 'Building...'
		    sh './gradlew assemble'
            }
        }

        def sonar() {
        stage('SonarQube Analysis') {
		    echo 'SonarQube...'
		    withSonarQubeEnv('SonarQube') {
		    sh './gradlew sonarqube -Dsonar.projectVersion=0.1'
		        }
            }
        }

        def artifactory() {
        stage('Publish Snapshot to Artifactory') {
		    echo 'Artifactory...'
            sh './gradlew publish'
                }
            }