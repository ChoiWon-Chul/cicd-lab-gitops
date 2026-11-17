pipeline {
  agent any
  triggers {
    pollSCM("H/5 * * * *")
  }
  environment {
    IMAGE = "localhost:5001/cicd-lab:jenkins-gitops-${BUILD_NUMBER}"
    GITOPS_REPO = "https://github.com/ChoiWon-Chul/cicd-lab-gitops.git"
    GITOPS_BRANCH = "codex/cicd-lab-gitops-manifests"
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
        writeFile file: "Dockerfile", text: "FROM nginx:alpine\\nCOPY index.html /usr/share/nginx/html/index.html\\n"
        sh "docker build -t ${IMAGE} ."
      }
    }
    stage("Push Registry") {
      steps {
        sh "docker push ${IMAGE}"
      }
    }
    stage("Update GitOps") {
      steps {
        withCredentials([string(credentialsId: "github-gitops-pat", variable: "GITHUB_TOKEN")]) {
          sh '''
            set -eu
            rm -rf gitops-update
            git -c http.extraheader="AUTHORIZATION: bearer ${GITHUB_TOKEN}" clone --branch "${GITOPS_BRANCH}" "${GITOPS_REPO}" gitops-update
            cd gitops-update
            sed -i.bak -E "s#image: .*#image: ${IMAGE}#" k8s/deployment.yaml
            rm -f k8s/deployment.yaml.bak
            git diff --check
            git config user.name "jenkins"
            git config user.email "jenkins@localhost"
            git add k8s/deployment.yaml
            git commit -m "chore: promote ${IMAGE}" 
            git -c http.extraheader="AUTHORIZATION: bearer ${GITHUB_TOKEN}" push origin "${GITOPS_BRANCH}"
            echo "GitOps commit pushed"
          '''
        }
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