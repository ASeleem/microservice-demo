pipeline {
  environment {
       registry = "aseleem/frontend"
       registryCredential = 'dockerhub_id'
       dockerImage = ''
       BUILD_NUMBER = '1.0'
   }
   agent {
        kubernetes {
            cloud 'k8s-cluster-016.aseleem.com'
            defaultContainer 'jnlp'
        }
    }
    stages {
        stage('Building and Delivering to DockerHub') {
            steps {
              podTemplate(yaml: '''
              apiVersion: v1
              kind: Pod
              spec:
                containers:
                - name: docker
                  image: docker:19.03.1-dind
                  securityContext:
                    privileged: true
                  env:
                    - name: DOCKER_TLS_CERTDIR
                      value: ""
              ''') {
                node(POD_LABEL) {
                    git 'https://github.com/GoogleCloudPlatform/microservices-demo.git'
                    container('docker') {
                      dir("${env.WORKSPACE}/src/frontend"){
                          script {
                              dockerImage = docker.build registry + ":$BUILD_NUMBER"
                          }
                      }
                      script {
                          docker.withRegistry( '', registryCredential ) {
                              dockerImage.push()
                          }
                      }
                    }
                }
              }
            }
        }
        stage('Deploying to K8') {
            steps {
              podTemplate(containers: [
                containerTemplate(name: 'golang', image: 'golang:1.16.5', command: 'sleep', args: '99d')
                ]) {
                  node(POD_LABEL) {
                    git 'https://github.com/ASeleem/microservice-demo.git'
                    sh 'ls -l'
                    container('golang') {
                      sh '''
                        curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                        install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
                        kubectl version --client
                        '''
                      withKubeConfig([credentialsId: 'k8s-prod-deploy-robot', serverUrl: 'https://k8s-master-01.aseleem.com:6443']) {
                        sh 'kubectl apply -f frontend.yaml -n production'
                      }
                    }
                  }
                }
            }
        }
    }
}
