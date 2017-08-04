pipeline {
  agent {
    node {
      label 'localhost_vagrant'
    }

  }
  stages {
    stage('Get Oracle Docker Sources') {
      steps {
        checkout([$class: 'GitSCM', branches: [[name: 'origin/master']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'CloneOption', depth: 0, noTags: true, reference: '', shallow: false, timeout: 30]], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'e7677693-d5e6-463f-b6d4-597e0fb91c3f', url: 'https://github.com/oracle/docker-images.git']]])
      }
    }
    stage('Build Oracle Docker Images') {
      steps {
        parallel(
          "Build Server JRE": {
            dir(path: 'OracleJava/java-8') {
              sh 'if [ ! -f $SW_FILE ]; then cp $SW_DIR/$SW_FILE" $SW_FILE; fi'
              sh 'sudo ./build.sh'
              sh 'docker tag oracle/serverjre:$SW_VERSION localhost:5000/oracle/serverjre:$SW_VERSION'
              sh 'docker push localhost:5000/oracle/serverjre:$SW_VERSION'
            }
          }
        )
      }
    }
    stage('Cleanup') {
      steps {
        parallel(
          "CleanUp Server JRE": {
            sh 'docker rmi --force localhost:5000/oracle/serverjre:$SW_VERSION'
            sh 'docker rmi --force oracle/serverjre:$SW_VERSION'
          }
        )
      }
    }

    environment {
      SW_VERSION = '8'
      SW_FILE = 'server-jre-8u144-linux-x64.tar.gz'
      SW_DIR = '/software/Oracle/Java'
    }
  }
}
