pipeline {
  parameters {
    string(name: 'tag', defaultValue: 'latest', description: 'tag {YYYYMMDD}{HHMMSS}')
  } 

  agent {
    kubernetes {
      yaml '''
apiVersion: v1
kind: Pod
spec:
  securityContext:
    runAsUser: 0
  containers:
  - name: shell
    image: ubuntu
    command:
    - sleep
    args:
    - infinity
  - name: docker
    image: docker:latest
    command:
    - cat
    tty: true
    volumeMounts:
    - mountPath: /var/run/docker.sock
      name: docker-sock
  - name: kubectl
    image: bitnami/kubectl:latest
    command:
    - sleep
    args:
    - infinity
    tty: true
  volumes:
    - name: docker-sock
      hostPath:
        path: /var/run/docker.sock
'''
       defaultContainer 'shell'
    }
  }

  environment {
    DOCKER_CREDENTIAL_ID = "wonkilee_dockerhub"
    K8S_CREDENTIAL_ID = "wonkilee_kubeconfig"
  }

  stages {
      
    stage('Example') {
      steps {
        echo 'Hello World!'
      }
    }

    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/chuirang/DevOps.git'
      }
    }
    
    stage('Docker build') {
      steps {
        container('docker') {
          script {
            env.IMAGE_TAG = "${params.tag}"
          }
          
          dir('docker') {
            withCredentials([usernamePassword(
              credentialsId: "${DOCKER_CREDENTIAL_ID}", // credentialsId
              usernameVariable: 'USERNAME', // 사용자명을 ${USERNAME} 환경변수에 mapping
              passwordVariable: 'PASSWORD'  // 사용자암호를 ${PASSWORD} 환경변수에 mapping
            )]) {
              sh "docker login -u ${USERNAME} -p ${PASSWORD}"
              sh "docker build -t ${USERNAME}/sampleapp:${env.IMAGE_TAG} ."
              sh "docker push ${USERNAME}/sampleapp:${env.IMAGE_TAG}"
            }
          }
        }
      }
    }

    stage('Kubernetes deploy') {
      steps {
        container('kubectl') {
          withCredentials([kubeconfigContent(credentialsId: "${K8S_CREDENTIAL_ID}", variable: 'KUBECONFIG_CONTENT')]) {
            sh '''echo "$KUBECONFIG_CONTENT" > kubeconfig'''
            sh 'kubectl --kubeconfig=kubeconfig apply -f k8s/deployment.yaml'
          }
        }
      }
    }
  }
}