pipeline {
    agent any
    tools {
        maven "maven"
    }
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "34.207.60.84:8081/"
        NEXUS_REPOSITORY = "Abdul"
        NEXUS_CREDENTIAL_ID = "My-Nexus"
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage("Clone Code") {
            steps {
                git 'https://github.com/betawins/sabear_simplecutomerapp.git'
            }
        }
        stage("Maven Build") {
            steps {
                sh 'mvn -Dmaven.test.failure.ignore=true clean install'
            }
        }
        stage("SonarCloud Analysis") {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=Ncodeit \
                        -Dsonar.projectName=Ncodeit \
                        -Dsonar.projectVersion=2.0 \
                        -Dsonar.sources=src/ \
                        -Dsonar.binaries=target/classes/ \
                        -Dsonar.junit.reportsPath=target/surefire-reports \
                        -Dsonar.jacoco.reportPath=target/jacoco.exec \
                        -Dsonar.java.binaries=src/
                    '''
                }
            }
        }
        stage("Publish to Nexus") {
            steps {
                script {
                    def pom = readMavenPom file: "pom.xml"
                    def filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    def artifactPath = filesByGlob[0].path
                    if (fileExists(artifactPath)) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version: ${pom.version}"
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [
                                    artifactId: pom.artifactId,
                                    classifier: '',
                                    file: artifactPath,
                                    type: pom.packaging
                                ],
                                [
                                    artifactId: pom.artifactId,
                                    classifier: '',
                                    file: "pom.xml",
                                    type: "pom"
                                ]
                            ]
                        )
                    } else {
                        error "*** File: ${artifactPath}, could not be found"
                    }
                }
            }
        }
        stage("Deploy to Tomcat") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'Tomcat-credentials', usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
                    script {
                        def warFile = sh(script: "ls target/*.war | head -n 1", returnStdout: true).trim()
                        echo "Deploying ${warFile} to Tomcat at context path /simplecustomerapp ..."
                        sh """
                            curl -u $TOMCAT_USER:$TOMCAT_PASS \
                                 -T ${warFile} \
                                 "http://34.229.166.230:8080/manager/text/deploy?path=/simplecustomerapp&update=true"
                        """
                    }
                }
            }
        }
        stage("Slack Notification") {
            steps {
                slackSend(
                    channel: env.SLACK_CHANNEL,
                    color: "#36A64F",
                    message: "Scripted pipeline for *Simple Customer App* has been successfully deployed in Tomcat :white_check_mark: by SNL for Job: ${env.JOB_NAME} [${env.BUILD_NUMBER}]"
                )
            }
        }
  }
}

