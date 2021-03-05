@Library('unifly-jenkins-common') _

def MAVEN_PARAMS = "-U -B -e -fae -V  -Dmaven.compiler.fork=true "

pipeline {
    agent any

    options {
        buildDiscarder logRotator(artifactNumToKeepStr: '5', numToKeepStr: '10')
        disableConcurrentBuilds()
    }

    environment {
        GIT_REPO = 'https://github.com/rgannu/camel-archetype-rabbitmq-kafka-connector.git'
        CREDENTIALS_ID = 'unifly-jenkins'
        JAVA_HOME="${tool 'openjdk-11'}"
        PATH="${env.JAVA_HOME}/bin:${tool 'nodejs-12'}/bin:${env.PATH}"
        UNIFLY_ARTIFACTORY = credentials('unifly-artifactory')
        artifactory_user = "$UNIFLY_ARTIFACTORY_USR"
        artifactory_password = "$UNIFLY_ARTIFACTORY_PSW"
    }

    stages {

        stage('Build & Package') {
            steps {
                sh "./mvnw $MAVEN_PARAMS clean package"
            }
        }

        stage('Deploy') {
            when { not { changeRequest() } }
            steps {
                  sh "./mvnw -s settings.xml $MAVEN_PARAMS deploy"
            }
        }
    }

    post {
        failure {
            sendSummary('#thirdparty')
        }
        fixed {
            sendSummary('#thirdparty')
        }

        always {
            archiveArtifacts artifacts: 'target/*.tar.gz,target/*.zip', onlyIfSuccessful:true, fingerprint: true
            // junit 'build/test-results/**/*.xml'
        }
    }
}
