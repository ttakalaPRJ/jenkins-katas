pipeline {
  agent any
  stages {
    stage('Clone Down') {
      agent { label 'swarm' }
      steps {
        stash name: "code", excludes: ".git"
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
            unstash "code"
            sh 'ci/build-app.sh'
            archiveArtifacts 'app/build/libs/'
          }
        }

        stage("Test App") {
          agent { docker { image "gradle:6-jdk11" } }
          options {
            skipDefaultCheckout()
          }
          steps {
            unstash "code"
            sh "ci/unit-test-app.sh"
            junit "app/build/test-results/test/TEST-*.xml"
          }
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