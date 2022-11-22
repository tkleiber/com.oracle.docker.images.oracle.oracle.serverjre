pipeline {

  agent {
    node {
      label 'docker_in_docker'
    }
  }

  environment {
    SW_VERSION = '8'
    # Search file in ENV JAVA_PKG in
    # https://github.com/oracle/docker-images/blob/main/OracleJava/8/Dockerfile.8
    # and download it as mentioned in the same file from
    # https://www.oracle.com/java/technologies/javase-server-jre8-downloads.html
    SW_FILE = 'server-jre-8u351-linux-x64.tar.gz'
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
