pipeline {
  agent any

    options {
      skipDefaultCheckout()
    }

  stages {

    stage('git clone') {
      agent {
        node {
          label 'Host'
        }
      }
      steps {
        checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/zanderhavgaard/jenkins-katas']]])
        stash excludes: '.git', name: 'code'
      }
    }

    stage('Parallel execution') {
      parallel {

        stage('Say hello!') {
          steps {
            sh 'echo "hello world!"'
          }
        }

        stage('build app') {
          agent {
            docker {
              image 'gradle:jdk11'
            }
          }

          steps {
            unstash 'code'
              sh 'bash ci/build-app.sh'
              archiveArtifacts 'app/build/libs/'
              deleteDir()
          }
        }

        stage('test app') {
          agent {
            docker {
              image 'gradle:jdk11'
            }
          }

          steps {
            unstash 'code'
              sh 'ci/unit-test-app.sh'
              junit 'app/build/test-results/test/TEST-*.xml'
              deleteDir()
          }
        }
      }
    }

  }

  post {
    always {

      deleteDir() /* clean up our workspace */
    }
  }

}
