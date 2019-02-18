def handleCheckout = {
    if (env.gitlabMergeRequestId) {
    	echo 'Merge request detected. Merging...'
    	def credentialsId = scm.userRemoteConfigs[0].credentialsId
    	checkout ([
    		$class: 'GitSCM',
    		branches: [[name: "${env.gitlabSourceNamespace}/${env.gitlabSourceBranch}"]],
    		extensions: [
    			[$class: 'PruneStaleBranch'],
    			[$class: 'CleanCheckout'],
    			[
    				$class: 'PreBuildMerge',
    				options: [
    					fastForwardMode: 'NO_FF',
    					mergeRemote: env.gitlabTargetNamespace,
    					mergeTarget: env.gitlabTargetBranch
    				]
    			]
    		],
    		userRemoteConfigs: [
    			[
    				credentialsId: credentialsId,
    				name: env.gitlabTargetNamespace,
    				url: env.gitlabTargetRepoSshURL
    			],
    			[
    				credentialsId: credentialsId,
    				name: env.gitlabSourceNamespace,
    				url: env.gitlabSourceRepoSshURL
    			]
    		]
    	])
    } else {
    	echo 'No merge request detected. Checking out current branch.'
    	checkout ([
    		$class: 'GitSCM',
    		branches: scm.branches,
    		extensions: [
    				[$class: 'PruneStaleBranch'],
    				[$class: 'CleanCheckout']
    		],
    		userRemoteConfigs: scm.userRemoteConfigs
    	    ])
        }
    }

pipeline {
    agent any


    stages {
	    stage('Merge?') {
            steps {
            script {
		    sh "env | sort"
		    handleCheckout()
		    sh "git branch -vv"
                }
	        }
        }
        stage('Build') {
            steps {
		    echo 'Building...'
		    sh './gradlew clean assemble'
            }
        }
        stage('Test') {
            steps {
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
        stage('SonarQube Analysis') {
            steps {
		    echo 'SonarQube...'
		    withSonarQubeEnv('SonarQube') {
		    sh './gradlew sonarqube -Dsonar.projectVersion=0.1'
		        }
            }
        }
        stage('Publish Snapshot to Artifactory') {
            steps {
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'jacocoexample-nexus-upload', usernameVariable: 'NEXUS_CREDENTIALS_USR', passwordVariable: 'NEXUS_CREDENTIALS_PSW']]) {
		    echo 'Artifactory...'
            sh './gradlew publish'
                }
            }
        }
    }
}