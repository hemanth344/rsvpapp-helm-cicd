pipeline {
    agent {
      kubernetes  {
            label 'jenkins-slave'
             defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: dind
    image: docker:18.09-dind
    securityContext:
      privileged: true
  - name: docker
    env:
    - name: DOCKER_HOST
      value: 127.0.0.1
    image: docker:18.09
    command:
    - cat
    tty: true
  - name: tools
    image: argoproj/argo-cd-ci-builder:v1.0.0
    command:
    - cat
    tty: true
"""
        }
    }
  environment {
      IMAGE_REPO = "hemanthhr/rsvpdummy"
      // Instead of nkhare, use your repo name
  }
  stages {
    stage('Build') {
      environment {
        DOCKERHUB_CREDS = credentials('dockerhub')
      }
      steps {
        container('docker') {
          sh "echo ${env.GIT_COMMIT}"
          // Build new image
          sh "until docker container ls; do sleep 3; done && docker image build -t  ${env.IMAGE_REPO}:${env.GIT_COMMIT} ."
          // Publish new image
          sh "docker login --username $DOCKERHUB_CREDS_USR --password $DOCKERHUB_CREDS_PSW && docker image push ${env.IMAGE_REPO}:${env.GIT_COMMIT}"
        }
      }
    }
    stage('Deploy to staging') {
      environment {
        GIT_CREDS = credentials('github')
        GIT_REPO_URL = "github.com/hemanth344/rsvpapp-helm-cicd.git"
        GIT_REPO_EMAIL = 'hemanth@cloudyuga.guru'
        GIT_REPO_BRANCH = "master"
       // Update above variables with your user details
      }
      steps {
        container('tools') {
            sh "git config --global user.email ${env.GIT_REPO_EMAIL}"
             // install wq
            sh "wget https://github.com/mikefarah/yq/releases/download/v4.9.6/yq_linux_amd64.tar.gz"
            sh "tar xvf yq_linux_amd64.tar.gz"
            sh "mv yq_linux_amd64 /usr/bin/yq"
            //install done
            sh '''#!/bin/bash
              echo $GIT_REPO_EMAIL
              echo $GIT_COMMIT
              yq eval '.image.repository = env(IMAGE_REPO)' -i rsvpapp-helm-cicd/package/values.yaml
              yq eval '.image.tag = env(GIT_COMMIT)' -i rsvpapp-helm-cicd/package/values.yaml
              cat rsvpapp-helm-cicd/package/values.yaml
            '''
            sh "git add rsvpapp-helm-cicd/package/values.yaml"
            sh "git commit -m 'Triggered Build'"
            sh "git push --set-upstream origin master"
            
        }
      }
    }   
  }
}

