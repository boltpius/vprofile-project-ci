pipeline {
    agent any 
    tools {
        jdk "JDK17"
        maven "MAVEN3.9"
    }

    environment {
        SNAP_REPO = 'vprofile-snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'admin123'
        RELEASE_REPO = 'vprofile-release'
        CENTRAL_REPO = 'vpro-maven-central'
        NEXUSIP = '172.31.43.144'
        NEXUSPORT = '8081'
        NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUS_LOGIN = 'nexuslogin'
        SONARSERVER = 'sonarserver'
        SONARSCANNER = 'sonarscanner'
    }

    stages {
        stage ('Build') {
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }

            post {
                success {
                    echo "Now Archiving"
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }

        stage ('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage ('Checkstyle Analysis') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }


        stage('Sonar Analysis') {
            environment {
                scannerHome = tool "${SONARSCANNER}"  // storing the global variable in the local variable scannerHome 
            }
            steps {
               withSonarQubeEnv("${SONARSERVER}") {  // this is using the variable to pass the value stored in it as the sonar server name saved  under systems in jenkins. this also scans for the unit test report and the checkstyle reports as well. SCans the code and takes the report to the sonar server.
                   sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
              }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("UploadArtifact") {
            steps {
                nexusArtifactUploader(
                            nexusVersion: 'nexus3', //the nexus current version 
                            protocol: 'http', //protocol used 
                            nexusUrl: "${NEXUSIP}:${NEXUSPORT}", //url to access your nexus server.
                            groupId: 'QA',
                            version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}", // the version to give the artifact being built
                            repository: "${RELEASE_REPO}", // the release repo on nexus to store the artifact 
                            credentialsId: "${NEXUS_LOGIN}", // credientials saved in jenkins for nexus access 
                            artifacts: [
                                [artifactId: 'vproapp', // artifact name
                                classifier: '',
                                file: 'target/vprofile-v2.war', //artifact you want to upload 
                                type: 'war'] 
                            ]
                        )
            }
        }

    }
}

