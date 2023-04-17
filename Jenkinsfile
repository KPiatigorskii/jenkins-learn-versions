pipeline {
    agent any

    environment {
        MAVEN_HOME = tool 'maven-tool'
        PATH = "$PATH:${MAVEN_HOME}/bin"
    }

    stages {
        stage('Calculate & Set Version') {
            when {
                expression { "${scm.branches[0].name}" =~ /^release\// }
            }

            steps {
                // sh "mvn build-helper:parse-version versions:set -DnewVersion='\${parsedVersion.majorVersion}.\${parsedVersion.minorVersion}.\${parsedVersion.nextIncrementalVersion}-SNAPSHOT' versions:commit"
                script{
                    def branchName = "${scm.branches[0].name}"
                    def versionParts = "${branchName}".tokenize('/')[1].tokenize('.')
                    def major = versionParts[0]
                    def minor = versionParts[1]
                    def patch = versionParts[2]

                    sh "mvn versions:set -DnewVersion='${major}.${minor}.${patch}-SNAPSHOT' versions:commit"
                    echo "Set version to ${major}.${minor}.${patch}-SNAPSHOT"
                    def pomXml = readMavenPom file: 'pom.xml'
                    echo "pomXml: $pomXml"
                    nextVersion = pomXml.version
                    sh "git commit -am 'Set version to ${nextVersion}'"
                    //sh "git push"
                }
            }       
        }
        stage('Build & Test') {
            steps {
                 withMaven(maven: 'maven-tool') {
                    sh 'mvn clean verify'
                 }
            }
        }
        stage('Checkstyle') {
            steps {
                withMaven(maven: 'maven-tool') {
                    sh 'mvn checkstyle:check'
                }
            }
        }
        stage('Publish') {
            when {
                expression { "${scm.branches[0].name}" == 'main' || "${scm.branches[0].name}" =~ /^release\// }
            }
            steps {
                withMaven(maven: 'maven-tool') {
                    sh 'mvn deploy'
                }
            }
        }
    }

    post {
        always {
            script {
                def currentBuildStatus = currentBuild.result

                if (currentBuildStatus == 'SUCCESS') {
                    slackSend(
                        color: "#00FF00",
                        channel: "jenkins-notify",
                        message: "${currentBuild.fullDisplayName} succeeded",
                        tokenCredentialId: 'slack-token'
                    )
                } else {
                    slackSend(
                        color: "#FF0000",
                        channel: "jenkins-notify",
                        message: "${currentBuild.fullDisplayName} was failed",
                        tokenCredentialId: 'slack-token'
                    )
                }
            }
        }
    }
}
