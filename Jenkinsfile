pipeline {
    agent any

    environment {
        MAVEN_HOME = tool 'maven-tool'
        PATH = "$PATH:${MAVEN_HOME}/bin"
    }

    stages {
        stage ('Increment snapshot'){            
            steps {
                        sh "mvn build-helper:parse-version versions:set /
                        -DnewVersion='\${parsedVersion.majorVersion}.\${parsedVersion.minorVersion}.\${parsedVersion.nextIncrementalVersion}-SNAPSHOT' versions:commit"
                }             
            }
        }
        stage('Calculate & Set Version') {
            when {
                expression { "${scm.branches[0].name}" =~ /^release\// }
            }




            steps {
                echo "BRANCH: ${scm.branches[0].name}"
                sh "ls"
                script {
                    def currentVersion = ''
                    def releaseVersion = ''
                    def nextVersion = ''

                    def pomXml = readMavenPom file: 'pom.xml'
                    echo "pomXml: $pomXml"
                    currentVersion = pomXml.version
                    echo "currentVersion: $currentVersion"
                    releaseVersion = "${currentVersion}".replaceAll(/-SNAPSHOT$/, '')
                    echo "releaseVersion: $releaseVersion"
                    nextVersion = "${releaseVersion}".tokenize('.').collect { it.toInteger() }.with {
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
                    echo "nextVersion: $nextVersion"
                    pomXml.version = nextVersion

                    writeMavenPom model: pomXml, file: 'pom.xml'
                    sh "git commit -am 'Set version to ${nextVersion}'"
                }
            }
        }
        stage('Build & Test') {
            steps {
                sh 'mvn clean verify'
            }
        }
        stage('Publish') {
            when {
                expression { "${scm.branches[0].name}" == 'main' || "${scm.branches[0].name}" =~ /^release\// }
            }
            steps {
                withMaven(maven: 'Maven') {
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
