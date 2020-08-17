pipeline {
  agent any

  environment {
    ARTIFACTORY_JENKINS_PASSWORD = credentials('artifactory-jenkins-password')
    DOCKER_HUB_USER = credentials('docker-hub-username')
    DOCKER_HUB_PASSWORD = credentials('docker-hub-password')
    GIT_COMMIT_HASH = "${GIT_COMMIT[0..7]}"
    GIT_BRANCH_NAME = "${GIT_BRANCH.minus(~/origin\\//)}"
    GIT_TARGET_BRANCH = "${GIT_BRANCH_NAME.minus(~/release.*-/) ==~ /^(develop|^[0-9\\.]+)$/ ? "${GIT_BRANCH_NAME}" : "master"}"
    IMAGE_TAG = "${BUILD_NUMBER}_${GIT_COMMIT_HASH}${GIT_BRANCH_NAME ==~ /release.*/ ? "_${GIT_BRANCH_NAME.minus(~/release.*-/)}" : ''}"
    SERVICE_NAME = 'twitter_wall_frontend'
    TOKEN = credentials('twitter-wall-server-token')
    JENKINSBOT_TOKEN = credentials('jenkinsbot-token')
    IS_RELEASE = "${GIT_BRANCH_NAME ==~ /release.*/ ? "yes" : "no"}"
  }

  triggers {
    pollSCM '* * * * *'
  }

  stages {
    stage('Install a packages') {
      steps {
          echo 'Installing...'
          sh 'npm install'
      }
    }
    stage('Build application') {
      steps {
        echo 'Building...'
        sh 'npm run build'
      }
    }
    stage('Build Docker image') {
      steps {
        echo 'Building Docker image...'
        sh "docker build -t naridre/twitter-wall-front:${IMAGE_TAG} ."
      }
    }
    stage('Push Docker images') {
      steps {
        echo 'Pushing Docker image...'
        sh '''
            docker login -u=$DOCKER_HUB_USER -p=$DOCKER_HUB_PASSWORD
            docker push naridre/twitter-wall-front:$IMAGE_TAG
        '''
      }
    }
    stage('Trigger DevOps upgrade version pipeline') {
      // when { expression { env.GIT_BRANCH_NAME ==~ /release.*/ } }
      steps {
        echo 'Triggering Jenkins DevOps pipeline...'
        sh "curl --user jenkinsbot:$JENKINSBOT_TOKEN -X POST 'http://localhost:8080/job/twitter-wall-devops/buildWithParameters?token=$TOKEN&TARGET_BRANCH=$GIT_TARGET_BRANCH&SERVICE_NAME=$SERVICE_NAME&SERVICE_VERSION=$IMAGE_TAG&COMMIT=$GIT_COMMIT_HASH'"
      }
    }
  }
}
