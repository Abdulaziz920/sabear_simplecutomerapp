pipeline {
    agent {
        label "master"
    }

    tools {
        maven "MVN_HOME"        // Maven tool configured in Jenkins
        // SonarQube CLI must be installed or configured under Global Tools as 'sonar-scanner'
    }

    environment {
        // Nexus configuration
        NEXUS_VERSION      = "nexus3"
        NEXUS_PROTOCOL     = "http"
        NEXUS_URL          = "34.207.60.84:8081/"
        NEXUS_REPOSITORY   = "Jenkins-04-Task-1"
        NEXUS_CREDENTIAL_ID= "Nexux_server"

        // Tomcat credentials
        TOMCAT_CRED        = "Tomcat-credentials"
    }

    stages {
        stage("Clone Code") {
            steps {
                git 'https://github.com/Abdulaziz920/sabear_simplecutomerapp.git'
            }
        }

        stage("Maven Build") {
            steps {
                sh 'mvn -Dmaven.test.failure.ignore=true install'
            }
        }

        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv('sonar-scanner') {  // Must match the name configured in Jenkins
                    sh 'sonar-scanner'
                }
            }
        }

        stage("Publish to Nexus") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml"
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    artifactPath = filesByGlob[0].path
                    artifactExists = fileExists artifactPath

                    if (artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}"
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId, classifier: '', file: artifactPath, type: pom.packaging],
                                [artifactId: pom.artifactId, classifier: '', file: "pom.xml", type: "pom"]
                            ]
                        )
                    } else {
                        error "*** File: ${artifactPath} could not be found"
                    }
                }
            }
        }

        stage("Deploy to Tomcat") {
            steps {
                withCredentials([usernamePassword(credentialsId: "${TOMCAT_CRED}", usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
                    sh '''
                        WAR_FILE=$(ls target/*.war | head -n 1)
                        WAR_NAME=$(basename $WAR_FILE .war | tr '[:upper:]' '[:lower:]')
                        echo "Deploying $WAR_FILE to Tomcat at context path /$WAR_NAME..."
                        curl -u $TOMCAT_USER:$TOMCAT_PASS \
                             -T $WAR_FILE \
                  "http://34.229.166.230:8080/manager/text/deploy?path=/$WAR_NAME&update=true"
                    '''
                }
            }
        }
    }
}
