pipeline {
  agent any
  environment {
    ORG = 'linkt'
    APP_NAME = 'newstart3'
    CHARTMUSEUM_CREDS = credentials('jenkins-x-chartmuseum')
    DOCKER_REGISTRY_ORG = 'linkt'
  }
  stages {
    stage('CI Build and push snapshot') {
      when {
        branch 'PR-*'
      }
      environment {
        PREVIEW_VERSION = "0.0.0-SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER"
        PREVIEW_NAMESPACE = "$APP_NAME-$BRANCH_NAME".toLowerCase()
        HELM_RELEASE = "$PREVIEW_NAMESPACE".toLowerCase()
      }
      steps {
        dir('/home/jenkins/go/src/github.com/linkt/newstart3') {
          checkout scm
          sh "make linux"
          sh "export VERSION=$PREVIEW_VERSION && skaffold build -f skaffold.yaml"
          sh "jx step post build --image $DOCKER_REGISTRY/$ORG/$APP_NAME:$PREVIEW_VERSION"
        }
        dir('/home/jenkins/go/src/github.com/linkt/newstart3/charts/preview') {
          sh "make preview"
          sh "jx preview --app $APP_NAME --dir ../.."
        }
      }
    }
    stage('Build Release') {
      when {
        branch 'master'
      }
      steps {	
        dir('/home/jenkins/go/src/github.com/linkt/newstart3') {
          git 'https://github.com/linkt/newstart3.git'

          // so we can retrieve the version in later steps
          sh "echo \$(jx-release-version) > VERSION"
          sh "jx step tag --version \$(cat VERSION)"
          sh "make build"
          sh "export VERSION=`cat VERSION` && skaffold build -f skaffold.yaml"
          sh "jx step post build --image $DOCKER_REGISTRY/$ORG/$APP_NAME:\$(cat VERSION)"
        }
      }
    }
    stage('Promote to Environments') {
    
      when {
        branch 'master'
      }
      steps {
        dir('/home/jenkins/go/src/github.com/linkt/newstart3/charts/newstart3') {
          sh "echo hello"
          sh "jx step changelog --version v\$(cat ../../VERSION)"

          // test
          sh "helm init --stable-repo-url http://mirror.azure.cn/kubernetes/charts/"

          // add repo
          sh "helm repo add storage.googleapis.com http://mirror.azure.cn/kubernetes/charts/"

          // release the helm chart
          sh "jx step helm release"

          // promote through all 'Auto' promotion Environments
          sh "jx promote -b --all-auto --timeout 1h --version \$(cat ../../VERSION)"
        }
      }
    }
  }
}
