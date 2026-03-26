# 🔄 cicd-pipeline-templates

> Ready-to-use CI/CD pipeline templates for Jenkins, GitHub Actions, and GitLab CI — covering build, test, Docker push, and Kubernetes deploy stages.

---

## 📁 Folder Structure

```
cicd-pipeline-templates/
├── jenkins/
│   ├── Jenkinsfile-basic
│   ├── Jenkinsfile-docker
│   └── Jenkinsfile-k8s-deploy
├── github-actions/
│   ├── build-and-push.yml
│   ├── deploy-to-eks.yml
│   └── terraform-apply.yml
├── gitlab-ci/
│   ├── .gitlab-ci-basic.yml
│   └── .gitlab-ci-docker-k8s.yml
└── README.md
```

---

## 🔵 Jenkins Pipeline (Jenkinsfile-k8s-deploy)

```groovy
pipeline {
    agent any

    environment {
        IMAGE_NAME   = "samantwanjare/myapp"
        IMAGE_TAG    = "${BUILD_NUMBER}"
        REGISTRY     = "docker.io"
        K8S_NS       = "production"
    }

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }
        stage('Build & Test') {
            steps {
                sh 'mvn clean test'
            }
        }
        stage('Docker Build') {
            steps {
                sh "docker build -t ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }
        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub', 
                                 usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh "echo $PASS | docker login -u $USER --password-stdin"
                    sh "docker push ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }
        stage('Deploy to K8s') {
            steps {
                sh """
                    kubectl set image deployment/myapp \
                    myapp=${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} \
                    -n ${K8S_NS}
                    kubectl rollout status deployment/myapp -n ${K8S_NS}
                """
            }
        }
    }
    post {
        success { echo "✅ Deployment Successful — Build #${BUILD_NUMBER}" }
        failure { echo "❌ Pipeline Failed — Check logs" }
    }
}
```

---

## 🟢 GitHub Actions (deploy-to-eks.yml)

```yaml
name: Build & Deploy to EKS

on:
  push:
    branches: [main]

env:
  AWS_REGION: ap-south-1
  ECR_REPOSITORY: myapp
  EKS_CLUSTER: prod-cluster

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1

      - name: Build, Tag & Push Docker image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG

      - name: Update kubeconfig
        run: |
          aws eks update-kubeconfig \
            --name ${{ env.EKS_CLUSTER }} \
            --region ${{ env.AWS_REGION }}

      - name: Deploy to EKS
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          kubectl set image deployment/myapp \
            myapp=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG \
            -n production
          kubectl rollout status deployment/myapp -n production
```

---

## 🟠 GitLab CI (.gitlab-ci-docker-k8s.yml)

```yaml
stages:
  - build
  - test
  - dockerize
  - deploy

variables:
  IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA

build:
  stage: build
  script:
    - mvn clean package -DskipTests

test:
  stage: test
  script:
    - mvn test

docker-build:
  stage: dockerize
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $IMAGE .
    - docker push $IMAGE

deploy-production:
  stage: deploy
  environment: production
  only:
    - main
  script:
    - kubectl set image deployment/myapp myapp=$IMAGE -n production
    - kubectl rollout status deployment/myapp -n production
```

---

## 🏷️ Tech Stack

![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=flat-square&logo=jenkins)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat-square&logo=github-actions)
![GitLab CI](https://img.shields.io/badge/GitLab_CI-FC6D26?style=flat-square&logo=gitlab)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker)
![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=flat-square&logo=kubernetes)

---

> 💡 **Author:** Samant Wanjare | [GitHub](https://github.com/samantwanjare) | [LinkedIn](https://linkedin.com/in/samantwanjare)
# cicd-pipeline-templates-
a
