pipeline {
    agent any

    environment {
        NEXUS_CRED = 'Nexux_server'          // Fixed typo
        TOMCAT_CRED = 'Tomcat-credentials'
    }

    stages {

        // -----------------------
        stage('Checkout SCM') {
            steps {
                git url: 'https://github.com/Abdulaziz920/hiring-app.git', branch: 'main'
            }
        }

        // -----------------------
        stage('Build') {
            steps {
                script {
                    // Set Maven from tool configuration
                    def mvnHome = tool name: 'MVN_HOME', type: 'maven'
                    env.PATH = "${mvnHome}/bin:${env.PATH}"
                }
                sh 'mvn clean package -DskipTests'
            }
        }

        // -----------------------
        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'Sonar-Scanner', variable: 'SONAR_TOKEN')]) {
                    withSonarQubeEnv('sonar-scanner') {
                        sh 'mvn sonar:sonar -Dsonar.token=$SONAR_TOKEN'
                    }
                }
            }
        }

        // -----------------------
        stage('Deploy to Nexus') {
            steps {
                script {
                    def mvnHome = tool name: 'MVN_HOME', type: 'maven'
                    env.PATH = "${mvnHome}/bin:${env.PATH}"
                }
                withCredentials([usernamePassword(credentialsId: "${NEXUS_CRED}", usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                    sh '''
                        mvn deploy -DskipTests -Dnexus.username=$NEXUS_USER -Dnexus.password=$NEXUS_PASS --settings /var/lib/jenkins/.m2/settings.xml
                    '''
                }
            }
        }

        // -----------------------
       stage('Deploy to Tomcat') {
    steps {
        withCredentials([usernamePassword(credentialsId: "Tomcat-credentials", usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
            sh '''
                # Get the WAR file
                WAR_FILE=$(ls target/*.war | head -n 1)
                if [ ! -f "$WAR_FILE" ]; then
                    echo "WAR file not found!"
                    exit 1
                fi

                # Generate context path from WAR name
                WAR_NAME=$(basename "$WAR_FILE" .war | tr '[:upper:]' '[:lower:]')

                echo "Deploying $WAR_FILE to Tomcat at context path /$WAR_NAME..."

                # Deploy WAR using Tomcat manager
                curl -v --fail --show-error -u $TOMCAT_USER:$TOMCAT_PASS \
                     -T "$WAR_FILE" \
                     "http://34.229.166.230:8080/manager/text/deploy?path=/$WAR_NAME&update=true"
            '''
        }
    }
}


        // -----------------------
        stage('Slack Notification') {
            steps {
                slackSend(
                    channel: '#jenkins-integration',
                    color: currentBuild.currentResult == 'SUCCESS' ? 'good' : 'danger',
                    message: "Hi Team, Jenkins pipeline for *Declarative pipeline* finished! ✅\nDeployed by: Abdul Aziz"
                )
            }
        }
    }

    // -----------------------
    post {
        always {
            echo 'Pipeline finished'
        }
        failure {
            slackSend(
                channel: '#jenkins-integration',
                color: 'danger',
                message: "⚠️ Jenkins pipeline for *Declarative pipeline* failed! Please check."
            )
        }
    }
}
