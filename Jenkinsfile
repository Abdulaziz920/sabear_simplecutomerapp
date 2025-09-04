pipeline {
    agent { label "master" }

    tools {
        maven "MVN_HOME"       // Jenkins Maven tool
        // Ensure SonarQube CLI is installed as 'sonar-scanner'
    }

    environment {
        // Nexus
        NEXUS_VERSION       = "nexus3"
        NEXUS_PROTOCOL      = "http"
        NEXUS_URL           = "34.207.60.84:8081"
        NEXUS_REPOSITORY    = "Jenkins-04-Task-1"
        NEXUS_CREDENTIAL_ID = "Nexux_server"

        // Tomcat
        TOMCAT_CRED         = "Tomcat-credentials"
    }

    stages {

        stage("Clone Code") {
            steps {
                git url: 'https://github.com/Abdulaziz920/sabear_simplecutomerapp.git', branch: 'master'
            }
        }

        stage("Maven Build") {
            steps {
                sh 'mvn clean install -Dmaven.test.failure.ignore=true'
            }
        }

        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv('sonar-scanner') {
                    sh 'sonar-scanner'
                }
            }
        }

        stage("Publish to Nexus") {
            steps {
                script {
                    // Use Maven CLI to extract info from pom.xml (sandbox-safe)
                    def groupId    = sh(script: "mvn help:evaluate -Dexpression=project.groupId -q -DforceStdout", returnStdout: true).trim()
                    def artifactId = sh(script: "mvn help:evaluate -Dexpression=project.artifactId -q -DforceStdout", returnStdout: true).trim()
                    def version    = sh(script: "mvn help:evaluate -Dexpression=project.version -q -DforceStdout", returnStdout: true).trim()
                    def packaging  = sh(script: "mvn help:evaluate -Dexpression=project.packaging -q -DforceStdout", returnStdout: true).trim()

                    def filesByGlob = findFiles(glob: "target/*.${packaging}")
                    if (filesByGlob.size() == 0) {
                        error "No artifact found in target directory"
                    }

                    def artifactPath = filesByGlob[0].path
                    echo "*** File: ${artifactPath}, group: ${groupId}, packaging: ${packaging}, version: ${version}"

                    nexusArtifactUploader(
                        nexusVersion: NEXUS_VERSION,
                        protocol: NEXUS_PROTOCOL,
                        nexusUrl: NEXUS_URL,
                        groupId: groupId,
                        version: version,
                        repository: NEXUS_REPOSITORY,
                        credentialsId: NEXUS_CREDENTIAL_ID,
                        artifacts: [
                            [artifactId: artifactId, classifier: '', file: artifactPath, type: packaging],
                            [artifactId: artifactId, classifier: '', file: "pom.xml", type: "pom"]
                        ]
                    )
                }
            }
        }

        stage("Deploy to Tomcat") {
            steps {
                withCredentials([usernamePassword(credentialsId: TOMCAT_CRED, usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
                    sh '''
                        WAR_FILE=$(ls target/*.war | head -n 1)
                        if [ ! -f "$WAR_FILE" ]; then
                            echo "No WAR file found in target directory"
                            exit 1
                        fi

                        WAR_NAME=$(basename $WAR_FILE .war | tr '[:upper:]' '[:lower:]')
                        echo "Deploying $WAR_FILE to Tomcat at context path /$WAR_NAME..."

                        curl -s -u $TOMCAT_USER:$TOMCAT_PASS \
                             -T $WAR_FILE \
                             "http://34.229.166.230:8080/manager/text/deploy?path=/$WAR_NAME&update=true" || exit 1
                    '''
                }
            }
        }
    }
}
