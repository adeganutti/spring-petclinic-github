pipeline {
    agent any

    environment {
            NEXUS_VERSION = "nexus3"
            NEXUS_PROTOCOL = "http"
            NEXUS_URL = "10.3.1.6:8081"
            NEXUS_REPOSITORY = "emerasoft-maven-nexus-repo"
            NEXUS_CREDENTIAL_ID = "sonatype-jenkins-user"
    }


    stages {
        stage('Maven Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
        }
        stage('Maven Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('JaCoCo Analysis') {
            steps {
                jacoco(execPattern: 'target/jacoco.exec')
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('Sonarqube') {
                  sh "mvn clean verify sonar:sonar -Dsonar.login=jenkins -Dsonar.password=jenkins"
                }
            }
        }
        stage("Nexus IQ Analysis") {
            steps {
                nexusPolicyEvaluation advancedProperties: '', enableDebugLogging: false, failBuildOnNetworkError: false, iqApplication: selectedApplication('spring-petclinic'), iqStage: 'build', jobCredentialsId: ''
            }
        }
        stage("Maven Package") {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }

        stage("Artifactory Repository Push") {
            steps{
                script {
                    rtServer (
                        id: 'artifactory-server-saas',
                        url: 'https://springpetclinictestpro.jfrog.io/',
                        // If you're using username and password:
                        username: 'jenkins',
                        password: 'Admin1234',
                        // If you're using Credentials ID:
                        //credentialsId: 'ccrreeddeennttiiaall',
                        // If Jenkins is configured to use an http proxy, you can bypass the proxy when using this Artifactory server:
                        bypassProxy: true,
                        // Configure the connection timeout (in seconds).
                        // The default value (if not configured) is 300 seconds:
                        timeout: 300
                    )

                    pom = readMavenPom file: "pom.xml"
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path
                    artifactExists = fileExists artifactPath
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}"
                    }

                    rtUpload (
                        serverId: 'artifactory-server-saas',
                        spec: '''{
                              "files": [
                                {
                                  "pattern": "target/*.jar",
                                  "target": "artifactory/example-repo-local/"
                                }
                             ]
                        }''',

                        // Optional - Associate the uploaded files with the following custom build name and build number,
                        // as build artifacts.
                        // If not set, the files will be associated with the default build name and build number (i.e the
                        // the Jenkins job name and number).
                        buildName: 'springpetclinic',
                        //buildNumber: '42',
                        // Optional - Only if this build is associated with a project in Artifactory, set the project key as follows.
                        project: 'spc'
                    )
                }
            }
        }

        /*stage("Nexus Repository Push") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }*/
    }
    post {
        always {
            junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'
        }
    }
}
