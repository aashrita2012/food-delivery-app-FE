node('master') {
  stage('Clone Projects') {
    parallel([

      earnings: {

        dir('earnings-regression-test') {

          git(
            url: 'https://github.com/anyTV/earnings.git',
            credentialsId: 'Swaraj'
          )
          sh 'curl https://gist.githubusercontent.com/arnoldamparo/443d1c22e764e77c404d98dc77181b33/raw/c1cdd704654e9de6e31a56a7fb533e23a998ad76/earnings-new.diff | git apply -v'
          sh 'npm install'
          sh 'nohup grunt -f > earnings.log &'
          sh 'nohup node queue_server.js > queue_server.log &'
        }
      },
      earnings_payment_gateway: {

        dir('earnings-payment-gateway') {
          git(
            url: 'https://gitlab.com/anyTV/earnings-payment-gateway.git',
            credentialsId: 'Swaraj_Gitlab'
          )
          sh 'npm install'
          sh 'nohup grunt > earnings-payment-gateway.log &'
        }
      }
    ])
  }

  stage('Test') {
    dir('freedom.cid.testsuite') {
      git(
        url: 'https://gitlab.com/anyTV/qa/freedom.cid.testsuite.git',
        credentialsId: 'Swaraj_Gitlab'
      )
      sh 'curl https://gist.githubusercontent.com/arnoldamparo/b393aaad4a536d0c4a2284705347d2f8/raw/477a70ae6e17d5ce8e017a117b0adcf053dbd929/earnings-automation.diff | git apply -v'
      wrap([$class: 'Xvfb', additionalOptions: '', assignedLabels: '', autoDisplayName: true, debug: true, displayNameOffset: 0, installationName: 'Xvfb', parallelBuild: true, screen: '1024x758x24', timeout: 25]) {
        sh 'echo $DISPLAY'
        sh 'export DISPLAY=:1'
        sh 'mvn -Dtest=EarningsRunner test -Dcucumber.options="--tags @init,@login,@masspay_gen,@terminate"'
      }
    }

  }
}
pipeline {
  agent any

  tools {
    nodejs "Nodejs"
  }

  environment {
    DOCKER_REGISTRY = "docker.io"
    DOCKERHUB_CREDENTIALS = credentials('DOCKER_HUB_CREDENTIAL')
    VERSION = "${env.BUILD_ID}"
  }

  stages {

    stage('Install Dependencies') {
      steps {

        sh 'npm ci'
      }
    }

    stage('Build Project') {
      steps {
        // Build the Angular project
        sh 'npm run build'
      }
    }

    stage('Docker Build and Push') {
      steps {
        sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
        sh 'docker build -t veerendra1976/food-delivery-app-fe:${VERSION} .'
        sh 'docker push veerendra1976/food-delivery-app-fe:${VERSION}'
      }
    }

    stage('Cleanup Workspace') {
      steps {
        deleteDir()
      }
    }

    stage('Update Image Tag in GitOps') {
      steps {
        checkout scmGit(branches: [
          [name: '*/main']
        ], extensions: [], userRemoteConfigs: [
          [credentialsId: 'git-ssh', url: 'git@github.com:aashrita2012/deployment-folder.git']
        ])
        script {
          // Set the new image tag with the Jenkins build number
          sh ''
          '
          sed - i "s/image:.*/image: veerendra1976\\/food-delivery-app-fe:${VERSION}/"
          aws / angular - manifest.yml ''
          '

          sh 'git checkout main'
          sh 'git add .'
          sh 'git commit -m "Update image tag"'
          sshagent(['git-ssh']) {
            sh('git push')
          }
        }
      }
    }
  }

}
