pipeline {

  agent {
    node {
      label 'docker_in_docker'
    }
  }

  environment {
    SW_VERSION = '8'
    SW_FILE = 'server-jre-8u311-linux-x64.tar.gz'
    SW_DIR = '/software/Oracle/Java'
  }

  options {
    buildDiscarder logRotator(numToKeepStr: '1')
  }

  stages {

    stage('Get Oracle Docker Sources') {
      steps {
        dir ('oracle') {
          git branch: 'main', credentialsId: 'github_id', url: 'https://github.com/oracle/docker-images.git'
        }
        sh 'ls -la'
      }
    }

    stage('Build Oracle Docker Images') {
      steps {
        parallel(
          "Build Server JRE": {
            dir(path: 'oracle/OracleJava/8/') {
              sh 'if [ ! -f $SW_FILE ]; then cp "$SW_DIR/$SW_FILE" $SW_FILE; fi'
              sh './build.sh'
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
  }

}
