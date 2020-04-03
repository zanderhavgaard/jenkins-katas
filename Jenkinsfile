pipeline {
  agent any

    stages {

      stage('git clone') {
        agent {
          node {
            label 'Host'
          }
        }
        steps {
            stash excludes: '.git', name: 'code'
        }
      }

      stage('Parallel execution') {
        parallel {

          stage('Say hello!') {
            options {
              skipDefaultCheckout(true)
            }
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

            options {
              skipDefaultCheckout(true)
            }

            steps {
              unstash 'code'
              sh 'bash ci/build-app.sh'
              archiveArtifacts 'app/build/libs/'
              stash excludes: '.git', name: 'build_app'
            }
          }

          stage('test app') {
            agent {
              docker {
                image 'gradle:jdk11'
              }
            }

            options {
              skipDefaultCheckout(true)
            }

            steps {
              unstash 'code'
              sh 'ci/unit-test-app.sh'
              junit 'app/build/test-results/test/TEST-*.xml'
            }
          }
        }
      }

      stage('docker build') {
        environment {
          docker_username = 'zanderhavgaard'
          DOCKERCREDS = credentials('docker_login') //use the credentials just created in this stage
        }
        steps {
          unstash 'build_app' //unstash the repository code
          sh 'ci/build-docker.sh'
          sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin' //login to docker hub with the credentials above
          sh 'ci/push-docker.sh'
        }
      }

    }

  post {
    always {
      deleteDir() /* clean up our workspace */
    }
  }

}
