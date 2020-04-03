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
                deleteDir()
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
                deleteDir()
            }
          }
        }
      }

      stage('docker build') {
        environment {
          DOCKERCREDS = credentials('docker_login') //use the credentials just created in this stage
        }
        steps {
          unstash 'code' //unstash the repository code
          unarchive mapping: ['app/build/libs/' : 'app/build/libs/']
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
