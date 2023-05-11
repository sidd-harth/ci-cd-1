pipeline {
    agent any

    tools {
        maven 'M386'
    }
    environment {
        BUILD_VERSION = "build.${currentBuild.number}"
        //  CLIENT_ID = credentials('anypoint.platform.clientId')
        //   CLIENT_SECRET = credentials('anypoint.platform.clientSecret')
        //   ANYPOINT_PLATFORM_CREDS = credentials('anypoint.platform.account')
        //   ASSET_VERSION = "v1"
    }

    stages {

        stage('Create Release Branch') {
            steps {

                echo "Starting Create Release Branch..."
                sh "git checkout -b 'release-${env.BUILD_VERSION}'"
                // sh "mvn versions:set -DnewVersion='release-${env.BUILD_VERSION}'"
                sh "git branch"
                echo "Create Release Branch: ${currentBuild.currentResult}"
            }
            post {
                success {
                    echo "...Create Release Branch Succeeded for 'release-${env.BUILD_VERSION}': ${currentBuild.currentResult}"
                }
                unsuccessful {
                    echo "...Create Release Branch Failed for 'release-${env.BUILD_VERSION}': ${currentBuild.currentResult}"
                }
            }
        }




        stage('Build and Verify') {
            steps {
                sh 'mvn clean verify'
            }
            post {
                success {
                    echo "...Build and Verify Succeeded for 'release-${env.BUILD_VERSION}': ${currentBuild.currentResult}"
                }
                unsuccessful {
                    echo "...Build and Verify Failed for 'release-${env.BUILD_VERSION}}': ${currentBuild.currentResult}"
                    script {
                        dropLocalReleaseBranch()
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube22') {
                    sh 'mvn clean verify sonar:sonar  -Dsonar.projectKey=ci-cd-1  \
                                            -Dsonar.host.url=http://localhost:9000 \
                                            -Dsonar.login=sqp_17f8ac444530d4cb6593114d4d00bb60715a30a0 \
                                            -Dsonar.sources=src/'
                }
            }
        }

        stage('SonarQube Quality Gate') {
            steps {
                withSonarQubeEnv('SonarQube22') {
                    timeout(time: 2, unit: 'MINUTES') {
                        script {
                            waitForQualityGate abortPipeline: true
                        }
                    }
                }
            }
            post {
                success {
                    echo "...SonarQube Quality Gate Succeeded for 'release-${env.BUILD_VERSION}': ${currentBuild.currentResult}"
                }
                unsuccessful {
                    echo "...SonarQube Quality Gate Failed for 'release-${env.BUILD_VERSION}}': ${currentBuild.currentResult}"
                    script {
                        dropLocalReleaseBranch()
                    }
                }
            }
        }

        stage('Push Artefact to Exchange') {
            steps {
                sh 'mvn clean deploy -Pexchange'
            }
            post {
                success {
                    echo "...Push Artefact to Exchange Succeeded for 'release-${env.BUILD_VERSION}': ${currentBuild.currentResult}"
                }
                unsuccessful {
                    echo "...Push Artefact to Exchange Failed for 'release-${env.BUILD_VERSION}}': ${currentBuild.currentResult}"
                    script {
                        dropLocalReleaseBranch()
                    }
                }
            }
        }

        stage('Deploy to CH2') {
            steps {
                sh 'mvn clean deploy -DmuleDeploy'
            }
            post {
                success {
                    echo "...Deploy to CH2 Succeeded for 'release-${env.BUILD_VERSION}': ${currentBuild.currentResult}"
                }
                unsuccessful {
                    echo "...Deploy to CH2 Failed for 'release-${env.BUILD_VERSION}}': ${currentBuild.currentResult}"
                    script {
                        dropLocalReleaseBranch()
                    }
                }
            }
        }

//   stage('Push Artefact to Nexus') {
//     steps {
//       sh 'mvn clean deploy -Pnexus-snapshot'
//     }
//   }

        stage('Push Release Branch') {
            steps {
                script {
                    echo "Starting Push Release Branch..."
                    sh "mvn --batch-mode release:clean release:prepare release:perform "
                }
            }
            post {
                success {
                    echo "...Push Release Branch Succeeded for 'release-${env.BUILD_VERSION}': ${currentBuild.currentResult}"
                }
                unsuccessful {
                    echo "...Push Release Branch Failed for 'release-${env.BUILD_VERSION}}': ${currentBuild.currentResult}"
                    script {
                        dropLocalReleaseBranch()
                    }
                }
            }
        }




    }
}


void dropLocalReleaseBranch() {
    echo "Starting Drop Local Release Branch: release-${env.BUILD_VERSION} ..."
    sh "git checkout main"
    sh "git branch -d 'release-${env.BUILD_VERSION}'"
    echo "...Local Release Branch release-${env.BUILD_VERSION} Dropped"
}