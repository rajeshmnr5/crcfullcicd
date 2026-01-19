pipeline {
  agent any

  environment {
    IMAGE_TAG = "${BUILD_NUMBER}"
  }

  stages {

    stage("Checkout") {
      steps {
        git "https://github.com/rajeshmnr5/crcfullcicd.git"
      }
    }

    stage("Prepare Kaniko Manifest") {
      steps {
        sh '''
        sed "s|\\${TAG}|${IMAGE_TAG}|g" ci/kaniko.yaml > kaniko-build.yaml
        '''
      }
    }

    stage("Build & Push Image") {
      steps {
        sh '''
        kubectl delete pod kaniko-build --ignore-not-found=true
        kubectl apply -f kaniko-build.yaml

        kubectl wait pod/kaniko-build \
          --for=condition=Ready \
          --timeout=300s || true

        kubectl logs -f kaniko-build
        '''
      }
    }
  }

  post {
    always {
      sh "kubectl delete pod kaniko-build --ignore-not-found=true"
    }
  }
}
