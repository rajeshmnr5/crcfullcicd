pipeline {

  agent {
    kubernetes {
      yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: kubectl
    image: lachlanevenson/k8s-kubectl:v1.29.0
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
          sed 's|{{TAG}}|${TAG}|g' ci/kaniko.yaml > /tmp/kaniko.yaml
        """
      }
    }

    stage('Build & Push Image') {
      steps {
        container('kubectl') {
          sh """
            kubectl delete pod kaniko-build --ignore-not-found=true
            kubectl apply -f /tmp/kaniko.yaml
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
