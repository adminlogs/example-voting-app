pipeline {
  agent none
  stages {
    stage('worker build') {
      agent {
        docker {
          image 'maven:3.6.1-jdk-8-alpine'
          args '-v $HOME/.m2:/root/.m2'
        }

      }
      when {
        changeset '**/worker/**'
      }
      steps {
        echo 'Compiling worker app'
        dir(path: 'worker') {
          sh 'mvn compile'
        }

      }
    }

    stage('worker test') {
      agent {
        docker {
          image 'maven:3.6.1-jdk-8-alpine'
          args '-v $HOME/.m2:/root/.m2'
        }

      }
      when {
        changeset '**/worker/**'
      }
      steps {
        echo 'Running Unit Tets on worker app'
        dir(path: 'worker') {
          sh 'mvn clean test'
        }

      }
    }

    stage('worker package') {
      agent {
        docker {
          image 'maven:3.6.1-jdk-8-alpine'
          args '-v $HOME/.m2:/root/.m2'
        }

      }
      when {
        branch 'master'
        changeset '**/worker/**'
      }
      steps {
        echo 'Packaging worker app'
        dir(path: 'worker') {
          sh 'mvn package -DskipTests'
        }

      }
    }

    stage('worker docker-package') {
      agent any
      when {
        branch 'master'
        changeset '**/worker/**'
      }
      steps {
        echo 'Packaging worker app with docker'
        script {
          docker.withRegistry('https://registry-1.docker.io/v2/', 'dockerlogin')
          {
            def workerImage = docker.build("adelelearning/worker:v${env.BUILD_ID}", "./worker")
            workerImage.push()
            workerImage.push("${env.BRANCH_NAME}")
          }
        }

      }
    }

    stage('result build') {
      agent {
        docker {
          image 'node:8.16.0-alpine'
          args '--user root'
        }

      }
      when {
        changeset '**/result/**'
      }
      steps {
        echo 'Compiling result app'
        dir(path: 'result') {
          sh 'npm install'
        }

      }
    }

    stage('result test') {
      agent {
        docker {
          image 'node:8.16.0-alpine'
          args '--user root'
        }

      }
      when {
        changeset '**/result/**'
      }
      steps {
        echo 'Running Unit Tests on result application'
        dir(path: 'result') {
          sh 'npm test'
        }

      }
    }

    stage('result docker-package') {
      agent any
      when {
        branch 'master'
        changeset '**/result/**'
      }
      steps {
        echo 'Packaging result app with docker'
        script {
          docker.withRegistry('https://registry-1.docker.io/v2/', 'dockerlogin')
          {
            def workerImage = docker.build("adelelearning/result:v${env.BUILD_ID}", "./result")
            workerImage.push()
            workerImage.push("${env.BRANCH_NAME}")
          }
        }

      }
    }

    stage('vote build') {
      agent {
        docker {
          image 'python:2.7.16-slim'
          args '--user root'
        }

      }
      when {
        changeset '**/vote/**'
      }
      steps {
        echo 'Compiling vote app'
        dir(path: 'vote') {
          sh 'pip install -r requirements.txt'
        }

      }
    }

    stage('vote test') {
      agent {
        docker {
          image 'python:2.7.16-slim'
          args '--user root'
        }

      }
      when {
        changeset '**/vote/**'
      }
      steps {
        echo 'Running Unit Tests on vote application'
        dir(path: 'vote') {
          sh 'pip install -r requirements.txt'
          sh 'nosetests -v'
        }

      }
    }

    stage('vote docker-package') {
      agent any
      when {
        branch 'master'
        changeset '**/vote/**'
      }
      steps {
        echo 'Packaging vote app with docker'
        script {
          docker.withRegistry('https://registry-1.docker.io/v2/', 'dockerlogin')
          {
            def workerImage = docker.build("adelelearning/vote:v${env.BUILD_ID}", "./vote")
            workerImage.push()
            workerImage.push("${env.BRANCH_NAME}")
          }
        }

      }
    }

    stage('Deploy to Dev') {
      agent any
      when {
        branch 'master'
      }
      steps {
        sh 'docker-compose up -d'
      }
    }

  }
  post {
    always {
      echo 'Building multibranch mono pipeline for instabote app is completed..'
    }

    success {
      slackSend(channel: '#instavote-cd', message: "Build succedded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
    }

    failure {
      slackSend(channel: '#instavote-cd', message: "Build failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
    }

  }
}
