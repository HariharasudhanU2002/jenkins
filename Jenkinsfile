pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                 checkout scmGit(branches: [[name: params.BRANCH]], extensions: [], userRemoteConfigs: [[credentialsId: 'bbpass', url: params.REPO_URL]])
            }
        }
         stage('COMMIT-Details') {
                steps {
                    script {
                        env.COMMIT_ID = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
                        env.LAST_COMMIT_DATE = sh(script: 'git log -1 --format=%cd --date=iso', returnStdout: true).trim()
                        env.REPO_NAME = sh(script: 'basename `git config --get remote.origin.url` .git', returnStdout: true).trim()
                        env.REPO_URL = sh(script: 'git config --get remote.origin.url', returnStdout: true).trim()

                        echo "Commit ID: ${env.COMMIT_ID}"
                        echo "Last Commit Date: ${env.LAST_COMMIT_DATE}"
                        echo "Repository Name: ${env.REPO_NAME}"
                        echo "Repository URL: ${env.REPO_URL}"
                }
            }
        }
        stage('Build') {
            tools {
                jdk 'java-17'
            }
            steps {
                sh 'chmod -R 755 .'
                sh 'mvn clean package'
            }
        }
        stage('Upload to Artifactory') {
          steps {
             archiveArtifacts artifacts: 'target/spring-petclinic-3.3.0-SNAPSHOT.jar', followSymlinks: false      
        }
    }
        stage('SonarQube analysis') {
            tools {
                jdk 'java-17'
            }
            environment {
                scannerHome = tool 'SonarQubeScanner'
                projectName = "Springboot-petclini"
            }
            steps {
                withSonarQubeEnv('sonar_1') {
                    sh """
                        export JAVA_HOME=\$JAVA_HOME
                        export PATH=\$JAVA_HOME/bin:\$PATH
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=${projectName} \
                        -Dsonar.sources=src \
                        -Dsonar.java.binaries=.
                    """
                }
            }
        }
        stage('Publish') {
            steps {
              nexusArtifactUploader artifacts: [[artifactId: 'Springboot-petclinic', classifier: '', file: '/var/lib/jenkins/workspace/Azure-Deployement/Release/target/spring-petclinic-3.3.0-SNAPSHOT.jar', type: 'tar']], credentialsId: 'nexusid', groupId: 'com.logicfocus', nexusUrl: '103.99.149.57:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'Springboot-petclinic', version: '1.0.1'
            }
        }
        stage('Docker Build Image') {
                 steps {
                     script { 
                    echo "Building Docker image..."
                    sh "docker build -t springboot-petclinic ."
                }
            }
        }
        stage('Build Info') {
            environment {
                NEXUS_URL = 'http://103.99.149.57:8081/'
                REPO_NAME = 'Springboot-petclinic'
                GROUP_ID = 'com.logicfocus'
                ARTIFACT_ID = 'Springboot-petclinic'
                VERSION = '1.0.1'
            }
            steps {
                script {
                    def artifactPath = "/${REPO_NAME}/${GROUP_ID.replace('.', '/')}/${ARTIFACT_ID}/${VERSION}/${ARTIFACT_ID}-${VERSION}.tar"
                    def nexusChecksumUrl = "${NEXUS_URL}/service/rest/v1/components?repository=${REPO_NAME}&group=${GROUP_ID}&name=${ARTIFACT_ID}&version=${VERSION}"

                    def artifactInfo = sh(script: "curl -s -u admin:lf@2024 ${nexusChecksumUrl}", returnStdout: true).trim()
                    def checksumValue = readJSON(text: artifactInfo).items[0].assets[0].checksum.sha1

                    def commitId = sh(script: "git rev-parse HEAD", returnStdout: true).trim()

                    sh """
                        echo '{' > buildData.json
                        echo '  "buildnumber": "${BUILD_NUMBER}",' >> buildData.json
                        echo '  "buildurl": "${JOB_URL}",' >> buildData.json
                        echo '  "group": "${GROUP_ID}",' >> buildData.json
                        echo '  "checksum": "${checksumValue}",' >> buildData.json
                        echo '  "artifact": "${ARTIFACT_ID}",' >> buildData.json
                        echo '  "ext": "tar",' >> buildData.json
                        echo '  "version": "${VERSION}",' >> buildData.json
                        echo '  "commitId": "${commitId}"' >> buildData.json
                        echo '}' >> buildData.json
                    """
                    stash name: 'buildDataStash', includes: 'buildData.json'
                }
            }
        }
        stage('Read Build Info') {
            steps {
                script {
                    unstash 'buildDataStash'
                    def buildData = readFile('buildData.json')
                    echo "Build Data: ${buildData}"

                    archiveArtifacts artifacts: 'buildData.json', onlyIfSuccessful: false
                }
            }
        }
    }
}
