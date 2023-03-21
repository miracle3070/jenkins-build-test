pipeline {
    agent any

    stages {
        stage('Install Dependencies') {
            steps {
                script {
                    // Alpine-based systems
                    def installCurl = "apk update && apk add curl"
                    def installJq = "apk add jq"

                    sh(installCurl)
                    sh(installJq)
                }
            }
        }

        stage('Check if commit was built') {
            steps {
                script {
                    // Jenkins configuration
                    def jenkins_url = "http://172.20.245.32:8080"
                    def job_name = "repository-build-test"

                    // Get the current build hash
                    def current_build_hash = sh(script: "git rev-parse HEAD", returnStdout: true).trim()

                    // Get the previous commit hashes (e.g., the last 10 commits)
                    def previous_commit_hashes = sh(script: "git log --pretty=format:'%H' -n 10", returnStdout: true).trim().split('\n')

                    // Function to check if a commit hash was built by Jenkins
                    def commit_built_by_jenkins = { commit_hash ->
                        def builds

                        withCredentials([usernamePassword(credentialsId: 'jenkins-api-token', usernameVariable: 'jenkins_username', passwordVariable: 'jenkins_token')]) {
                            builds = sh(script: "curl -s --user \"\$jenkins_username:\$jenkins_token\" \"$jenkins_url/job/$job_name/api/json?tree=builds%5Bid%2Cresult%2Cactions%5BlastBuiltRevision%5BSHA1%5D%5D%5D&depth=2\"", returnStdout: true).trim()
                }

                        // Check if the commit hash appears in a build with a non-null result
                        def jq_query = ".builds[] | select(.actions[].lastBuiltRevision.SHA1==\"$commit_hash\" and .result!=null) | .result"
                        def result = sh(script: "echo '${builds}' | jq -r '${jq_query}' | grep -q .; echo \$?", returnStdout: true).trim()

                        return result == "0"
                    }

                    // Check if the current build hash exists in the previous commit hashes list and was built by Jenkins
                    def alreadyBuilt = false
                    for (commit_hash in previous_commit_hashes) {
                        if (commit_hash == current_build_hash && commit_built_by_jenkins(commit_hash)) {
                            echo "The current build hash ($current_build_hash) already exists in the previous commit hashes list and was built by Jenkins. Stopping the build."
                            alreadyBuilt = true
                            break
                        }
                    }

                    if (!alreadyBuilt) {
                        echo "The current build hash ($current_build_hash) does not exist in the previous commit hashes list or was not built by Jenkins. Continuing the build."
                    } else {
                        error("Build stopped because the commit was already built.")
                    }
                }
            }
        }

        // Add other stages for your build process here
    }
}