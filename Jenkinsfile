pipeline {
    agent {
        label "master"
    }

    tools {
        maven "MVN_HOME"        // Maven tool configured in Jenkins
        // Make sure SonarQube CLI is installed as 'sonar-scanner' under Global Tool Configuration
    }

    environment {
        // Nexus configuration
        NEXUS_VERSION       = "nexus3"
        NEXUS_PROTOCOL      = "http"
        NEXUS_URL           = "34.207.60.84:8081"
        NEXUS_REPOSITORY    = "Jenkins-04-Task-1"
        NEXUS_CREDENTIAL_ID = "Nexux_server"

        // Tomcat credentials
        TOMCAT_CRED         = "Tomcat-credentials"
    }

    stages {

        stage("Clone Code") {
            steps {
                git url: 'https://github.com/Abdulaziz920/sabear_simplecutomerapp.git', branch: 'main'
            }
        }

        stage("Maven Build") {
            steps {
                sh 'mvn clean install -Dmaven.test.failure.ignore=true'
            }
        }

        stage("SonarQube Analysis") {
            steps {
                // 'sonar-scanner' must exactly match the name in Jenkins Global Tool Configuration
                withSonarQubeEnv('sonar-scanner') {
                    sh 'sonar-scanner'
                }
            }
        }

        stage("Publish to Nexus") {
            steps {
                script {
                    def pom = readMavenPom file: "pom.xml"
                    def filesByGlob = findFiles(glob: "target/*.${pom.packaging}")

                    if (filesByGlob.size() == 0) {
                        error "No artifact found in target directory"
                    }

                    def artifactPath = filesByGlob[0].path
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
                            [artifactId: pom.artifactId, classifier: '', file: artifactPath, type: pom.packaging],
                            [artifactId: pom.artifactId, classifier: '', file: "pom.xml", type: "pom"]
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
