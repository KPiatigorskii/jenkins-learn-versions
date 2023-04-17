// Write a Jenkinsfile that performs the following steps:

// Calculate & Set Version: This stage is run when the branch name matches the pattern "release/*".
// It calculates the next version number for the release, sets the version number in the project's pom.xml file, and commits the change to the source code repository.

// Build & Test: This stage builds and tests the project.

// Publish: This stage is run when the branch name is either "main" or matches the pattern "release/*". 
// It deploys the project to a Maven repository using settings defined in a configuration file.


//  Add a slack notification
// which informs if the build succeeded or failed

// Bonus:
// Add Code analysis with 'checkstyle'

node {

    def currentVersion = ''
    def releaseVersion = ''
    def nextVersion = ''

    try {
        echo "${scm.getBranches()[0]}"
        if (env.BRANCH_NAME.startsWith('release/')){
            stage('Calculate & Set Version'){

                echo "Calculate & Set Version"
                def pomXml = readMavenPom file: 'pom.xml'
                currentVersion = pomXml.version
                releaseVersion = currentVersion.replaceAll(/-SNAPSHOT$/, '')
                nextVersion = releaseVersion.tokenize('.').collect { it.toInteger() }.with {
                    set(2, it[2] + 1)
                    if (it[2] >= 10) {
                        set(1, it[1] + 1)
                        set(2, 0)
                    }
                    if (it[1] >= 10) {
                        set(0, it[0] + 1)
                        set(1, 0)
                    }
                    join('.')
                }
                pomXml.version = nextVersion
                writeMavenPom model: pomXml, file: 'pom.xml'
                sh "git commit -am 'Set version to ${nextVersion}'"
            }
        }

        stage("Build & Test"){

        }
        stage("Publish"){

        }

            
    }
    catch (e) {
        stage('slack-notify'){
            slackSend(
                color: "#FF0000",
                channel: "jenkins-notify",
                message: "${currentBuild.fullDisplayName} was failed",
                tokenCredentialId: 'slack-token' 
            )
        }

    }
    finally{
        stage("summary"){
            echo "${env.BRANCH_NAME}"
            echo "currentVersion: ${currentVersion}"
            echo "releaseVersion: ${releaseVersion}"
            echo "nextVersion ${nextVersion}"
        }
        stage("check-code with checkstyle"){

        }
        stage('slack-notify'){
            slackSend(
                color: "#FF0000",
                channel: "jenkins-notify",
                message: "${currentBuild.fullDisplayName} succeeded",
                tokenCredentialId: 'slack-token' 
            )
        }
    }



}