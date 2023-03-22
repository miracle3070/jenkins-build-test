pipeline {
    agent any

    environment {
        GIT_URL = 'https://github.com/miracle3070/jenkins-build-test'
        JENKINS_URL = 'http://172.20.245.32:8080/'
        JOB_NAME = 'jenkins-build-test-pipeline'
        AUTHENTICATION_ID = 'jenkins_api_token123'
    }

    stages {
        stage('Check Commit Hash') {
            steps {
                script {
                    // Get the latest commit hash from the remote repository
                    def latestCommit = sh(script: "git ls-remote ${GIT_URL} HEAD | awk '{print \$1}'", returnStdout: true).trim()
                    echo "Latest commit hash: ${latestCommit}"

                    def buildStatusUrl = "${JENKINS_URL}/job/${JOB_NAME}/api/json?tree=allBuilds[result,actions[buildsByBranchName[*[*]]]]"
                    def buildStatusResponse = httpRequest(url: buildStatusUrl, authentication: AUTHENTICATION_ID, acceptType: 'APPLICATION_JSON')
                    def buildStatusJson = readJSON text: buildStatusResponse.content

                    def commitBuilt = false
                    for (build in buildStatusJson.allBuilds) {
                        def branchBuildInfo = build.actions.find { it.buildsByBranchName }
                        if (branchBuildInfo) {
                            def commitInfo = branchBuildInfo.buildsByBranchName.values().find { it.revision.SHA1 == latestCommit }
                            if (commitInfo && build.result == 'SUCCESS') {
                                commitBuilt = true
                                break
                            }
                        }
                    }

                    if (commitBuilt) {
                        echo "Latest commit ${latestCommit} was built successfully. Stopping the build."
                        currentBuild.result = 'ABORTED'
                        error("Build was aborted because the latest commit ${latestCommit} has already been built successfully.")
                    } else {
                        echo "Latest commit ${latestCommit} was not built or not built successfully. Continuing the build."
                    }
                }
            }
        }

        stage('Clone Repository') {
            steps {
                git branch: 'master', url: "${GIT_URL}"
            }
        }

        stage('Build') {
            steps {
                // Your build steps go here
                echo "Building the project..."
            }
        }

        stage('Test') {
            steps {
                // Your test steps go here
                echo "Testing the project..."
            }
        }

        stage('Deploy') {
            steps {
                // Your deploy steps go here
                echo "Deploying the project..."
            }
        }
    }
}