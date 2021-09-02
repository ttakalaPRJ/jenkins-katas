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
            archiveArtifacts 'app/build/libs/'
            stash includes: 'app/build/libs/*.jar', name: 'app'
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
      }
    }
      stage('Push app') {
        when { branch 'master' }
        environment {
            DOCKERCREDS = credentials('docker_login')
        }
        steps {
            unstash 'code'
            unstash 'app'
            sh 'ci/build-docker.sh'
            sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin'
            sh 'ci/push-docker.sh'
        }
      }
      stage('Component test') {
          when not { branch 'dev/*' }
          steps {
            sh 'ci/component-test.sh'
          }
      }
  }
  post {
    cleanup {
      deleteDir()
    }
  }
}
