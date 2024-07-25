pipeline {
  agent {
    kubernetes {
      yamlFile 'Jenkins-agent-pod.yaml'
      }
    }

  environment {
    DOCKERFILE_PATH = 'Dockerfile'
    DOCKER_IMAGE_NAME = 'myrepo/myapp:tag'
  }

  stages {

    stage('Build and Push Docker Image with Kaniko') {
      steps {
      container(name: 'kaniko', shell: '/busybox/sh') {
        withCredentials([file(credentialsId: 'dockerhub', variable: 'DOCKER_CONFIG_JSON')]) {
          withEnv(['PATH+EXTRA=/busybox']) {
            sh '''#!/busybox/sh
              cp $DOCKER_CONFIG_JSON /kaniko/.docker/config.json
              /kaniko/executor --context `pwd` --dockerfile $DOCKERFILE_PATH --destination $DOCKER_IMAGE_NAME
            '''
            }
          }
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
      container(name: 'kubectl', shell: '/bin/sh') {
        withCredentials([file(credentialsId: 'k3s-hq-admin', variable: 'KUBECONFIG')]) {
          sh "echo $KUBECONFIG > /.kube/config"
          sh "kubectl apply -f manifest.yaml"
          }
        }
      }
    }
  }
}
