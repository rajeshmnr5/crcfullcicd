pipeline {

  agent {
    kubernetes {
      yaml """
apiVersion: v1
kind: Pod
spec:
  serviceAccount: jenkins-sa
  containers:
  - name: kubectl
    image: dtzar/helm-kubectl:3.14.0
    command:
    - sh
    - -c
    - cat
    tty: true

  - name: jnlp
    image: jenkins/inbound-agent:latest
"""
    }
  }

  environment {
    IMAGE = "192.168.29.202:32001/crcfullcicd"
    TAG   = "${BUILD_NUMBER}"
  }

  stages {

    stage('Prepare Kaniko Manifest') {
      steps {
        sh """
          sed "s|{{TAG}}|${BUILD_NUMBER}|g" ci/kaniko.yaml > kaniko.yaml
        """
      }
    }

    stage('Build & Push Image') {
      steps {
        container('kubectl') {
          sh """
            kubectl delete pod kaniko-build --ignore-not-found=true
            kubectl apply -f kaniko.yaml
            kubectl wait pod/kaniko-build --for=condition=Succeeded --timeout=600s
          """
        }
      }
    }
  }

  post {
    always {
      container('kubectl') {
        sh "kubectl delete pod kaniko-build --ignore-not-found=true"
      }
    }
  }
}
