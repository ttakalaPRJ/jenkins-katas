pipeline {
  agent any
  environment {
        docker_username = 'terotakala'
  }
  stages {
    stage('Clone Down') {
      agent { label 'swarm' }
      steps {
        stash name: 'code', excludes: '.git'
      }
    }
    stage('Parallel execution') {
      parallel {
        stage('Hello') {
          steps {
            sh 'echo "hello world"'
          }
        }

        stage('Build App') {
          agent {
            docker {
              image 'gradle:6-jdk11'
            }
          }
          options {
            skipDefaultCheckout()
          }
          steps {
            unstash 'code'
            sh 'ci/build-app.sh'
            stash name: 'code', excludes: '.git'
            archiveArtifacts 'app/build/libs/'
          }
        }

        stage('Test App') {
          agent { docker { image 'gradle:6-jdk11' } }
          options {
            skipDefaultCheckout()
          }
          steps {
            unstash 'code'
            sh 'ci/unit-test-app.sh'
            junit 'app/build/test-results/test/TEST-*.xml'
          }
        }
        stage('somestage')
        environment {
          DOCKERCREDS = credentials('docker_login') //use the credentials just created in this stage
        }
        steps {
          unstash 'code' //unstash the repository code
          sh 'ci/build-docker.sh'
          sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin' //login to docker hub with the credentials above
          sh 'ci/push-docker.sh'
        }
      }
    }
  }
  post {
    cleanup {
      deleteDir()
    }
  }
}
