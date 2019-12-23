#!groovy



def label = "jenkins"
env.DOCKERHUB_IMAGE = "devops53/hello-world"


podTemplate(label: label, yaml: """
apiVersion: v1
kind: Pod
metadata:
  name: jenkins-slave
  namespace: jenkins
  labels:
    component: ci
    jenkins: jenkins-agent
spec:
  # Use service account that can deploy to all namespaces
  serviceAccountName: jenkins
  volumes:
  - name: dind-storage
    emptyDir: {}
  containers:
  - name: git
    image: alpine/git
    command:
    - cat
    tty: true
  - name: nodejs
    image: node:latest
    command:
    - cat
    tty: true
  - name: kubectl
    image: bitnami/kubectl:latest
    command:
    - cat
    tty: true
  - name: docker
    image: docker:19-git
    command:
    - cat
    tty: true
    env:
    - name: DOCKER_HOST
      value: tcp://docker-dind:2375
    volumeMounts:
      - name: dind-storage
        mountPath: /var/lib/docker
  - name: helm
    image: lachlanevenson/k8s-helm:latest
    command:
    - cat
    tty: true
"""
  )

  {

  node(label) {
    
    stage('Checkout SCM') {
        checkout scm
    } 

    stage('Build node.js app') {
        container('nodejs') {
        sh 'npm install'
    }
      }
    stage('Node.js test') {
        container('nodejs') {
        sh 'npm test'
    }
      }
    stage('Build docker image')
        container('docker') {

         if (env.BRANCH_NAME != 'master' && env.CHANGE_ID == null){
           return 0
          }


      echo "Docker build image name ${DOCKERHUB_IMAGE}:${BRANCH_NAME}"
           sh 'docker build . -t ${DOCKERHUB_IMAGE}:${BRANCH_NAME}'
            }



    container('docker') {
       withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')]){
            sh """
             docker login --username ${DOCKER_USER} --password ${DOCKER_PASSWORD}
             docker build . -t ${DOCKERHUB_IMAGE}:${BRANCH_NAME} 
             docker push ${DOCKERHUB_IMAGE}:${BRANCH_NAME}
            """
            }
        }

   
          
    }
  }