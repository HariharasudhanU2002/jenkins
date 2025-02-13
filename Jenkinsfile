pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                 checkout scmGit(branches: [[name: params.BRANCH]], extensions: [], userRemoteConfigs: [[credentialsId: 'spring', url: params.REPO_URL]])
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
                  sh 'mvn clean'
                  sh 'mvn compile'
                  sh 'mvn clean package -X'
               }   
         }
         stage('Upload to Artifactory') {
             steps {
                    archiveArtifacts artifacts: 'target/spring-petclinic-3.3.0-SNAPSHOT.jar', followSymlinks: false      
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
    }
}
