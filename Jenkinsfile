pipeline {
    agent {
        label 'linux'
    }

    triggers {
        pollSCM '@hourly'
        cron '@daily'
    }

    tools {
        maven 'Latest Maven'
    }

    options {
        ansiColor('xterm')
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '15')
        timestamps()
        disableConcurrentBuilds()
    }

    stages {
        stage('Build') {
           when {
             not {
               branch "master"
             }
           }

           steps {
               configFileProvider([configFile(fileId: '071964a6-1e15-418b-9a5d-4bd5cbdadeec', variable: 'MAVEN_SETTINGS_XML')]) {
                   sh label: 'maven package', script: 'mvn -B -s "$MAVEN_SETTINGS_XML" -DskipTests package'
               }
           }
        }
        stage('Deploy Java Module') {
            when {
                allOf {
                    branch "master"
                    not {
                        triggeredBy "TimerTrigger"
                    }
                }

            }

            steps {
                configFileProvider([configFile(fileId: '071964a6-1e15-418b-9a5d-4bd5cbdadeec', variable: 'MAVEN_SETTINGS_XML')]) {
                    sh label: 'maven deploy', script: 'mvn -B -s "$MAVEN_SETTINGS_XML" -DskipTests deploy'
                }
            }
        }
    }

    post {
        unsuccessful {
            mail to: "rafi@guengel.ch",
                    subject: "${JOB_NAME} (${BRANCH_NAME};${env.BUILD_DISPLAY_NAME}) -- ${currentBuild.currentResult}",
                    body: "Refer to ${currentBuild.absoluteUrl}"
        }
    }
}
