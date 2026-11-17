pipeline {
  agent any
  triggers {
    pollSCM("H/5 * * * *")
  }
  stages {
    stage("Checkout") {
      steps {
        checkout scm
        sh "git rev-parse --short HEAD"
      }
    }
    stage("Build Image") {
      steps {
        writeFile file: "index.html", text: "Docker Kubernetes Jenkins Argo CD"
        writeFile file: "Dockerfile", text: "FROM nginx:alpine\nCOPY index.html /usr/share/nginx/html/index.html\n"
        sh "docker build -t localhost:5001/cicd-lab:jenkins-scm ."
      }
    }
    stage("Push Registry") {
      steps {
        sh "docker push localhost:5001/cicd-lab:jenkins-scm"
      }
    }
    stage("Deploy Kubernetes") {
      steps {
        sh "echo Image pushed - Argo CD will reconcile the GitOps manifest"
      }
    }
    stage("Argo CD Sync") {
      steps {
        sh "echo Verify Argo CD Application is Synced and Healthy"
      }
    }
  }
}