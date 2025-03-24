## 📝 개요

Docsker

---

## 👥 팀원
| 이름       | GitHub                                  |
|------------|-----------------------------------------|
| 장현진     | [<img src="https://img.shields.io/badge/Github-Link-181717?logo=Github">](https://github.com/CoderJDan) |
| 남궁일     | [<img src="https://img.shields.io/badge/Github-Link-181717?logo=Github">](https://github.com/namgungil) |
| 이상엽     | [<img src="https://img.shields.io/badge/Github-Link-181717?logo=Github">](https://github.com/frenchfries0927) |
| 이효재     | [<img src="https://img.shields.io/badge/Github-Link-181717?logo=Github">](https://github.com/Alexandra) |
| 윤성훈     | [<img src="https://img.shields.io/badge/Github-Link-181717?logo=Github">](https://github.com/YunSHCode) |

---

## 🔧 기술 스택
| 분야          | 기술 스택                |
|---------------|--------------------------|
| **Front-End** | <img src="https://img.shields.io/badge/HTML-E34F26?style=for-the-badge&logo=HTML5&logoColor=white"> <img src="https://img.shields.io/badge/CSS-1572B6?style=for-the-badge&logo=CSS3&logoColor=white"> <img src="https://img.shields.io/badge/JavaScript-F7DE1E?style=for-the-badge&logo=JavaScript&logoColor=white"> |
| **Back-End**     | <img src="https://img.shields.io/badge/Java-007396?style=for-the-badge&amp;logo=Java&amp;logoColor=white" alt="Java Badge"/> <img src="https://img.shields.io/badge/Spring-6DB33F?style=for-the-badge&amp;logo=Spring&amp;logoColor=white" alt="Spring Badge" /> <img src="https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&amp;logo=MySQL&logoColor=white"/> |

---

## 🛠️ 개발 도구
| IDE          | 버전 관리                |
|---------------|--------------------------|
| <img src="https://img.shields.io/badge/IntelliJ IDEA-000000?style=for-the-badge&logo=IntelliJ IDEA&logoColor=white"> | <img src="https://img.shields.io/badge/Github-181717?style=for-the-badge&amp;logo=Github&amp;logoColor=white" alt="Github Badge" /> |

---

### 📌 ERD
![ERD](https://github.com/Docsker/Docsker_Build/blob/a587ed7639a0e0d0e6fcff9fb0b6a3c31a619fe9/public/architecture.png)

---


## 🚀 빌드 및 배포 개요

### 📌 사용 기술 및 도구

- **운영체제:** Ubuntu (가상 서버)
- **언어:** Java (Spring Boot)
- **빌드 도구:** Gradle
- **컨테이너화:** Docker
- **CI/CD 도구:** GitHub Actions, Jenkins, ArgoCD
- **배포 인프라:** Kubernetes
- **데이터베이스:** MySQL

### 🔄 CI/CD 개요

1. **GitHub**: 소스 코드 관리 및 CI/CD 트리거
2. **Jenkins**: 자동화된 빌드 및 Docker 이미지 생성, Docker Hub에 푸시
3. **Docker Hub**: 빌드된 이미지를 저장하고 Kubernetes에서 사용
4. **ArgoCD**: Kubernetes 클러스터에 자동 배포

---


## 🏗️ 빌드 프로세스
### Git Push
![ngrok](https://github.com/CoderJDan/getRand-build-release/blob/d8ba0a444c8859f52e576f33ba0286ff21dd4ea1/build_release_screenshot/ngrok.png?raw=true)

![webhook](https://github.com/CoderJDan/getRand-build-release/blob/master/build_release_screenshot/webhook.png?raw=true)


### 📂 Dockerfile을 이용한 빌드

```Dockerfile
# Build Stage
FROM gradle:8.11.1-jdk17 AS build

# 작업 디렉토리 생성
WORKDIR /myapp

# 프로젝트 전체 파일을 복사
COPY . /myapp

# Gradle 실행 권한 추가
RUN chmod +x /myapp/gradlew

# Gradle 빌드 실행 (테스트 제외)
RUN /myapp/gradlew clean build --no-daemon -x test

# Run Stage
FROM openjdk:17-alpine

# 작업 디렉토리 생성
WORKDIR /myapp

# 빌드된 JAR 파일 복사
COPY --from=build /myapp/build/libs/*SNAPSHOT.jar /myapp/getrand.jar

# 애플리케이션 실행 포트
EXPOSE 5679

# 애플리케이션 실행 명령어
ENTRYPOINT ["java", "-jar", "/myapp/getrand.jar"]
```

---

## ☸️ Kubernetes 배포

### 📜 배포 리소스 정의 (`getrand-datacollection-deploy.yaml, getrand-datacollection-service.yaml`)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: datacollectionservice
  namespace: getrand
  labels:
    service: datacollectionservice
spec:
  replicas: 2
  selector:
    matchLabels:
      service: datacollectionservice
  template:
    metadata:
      labels:
        service: datacollectionservice
    spec:
      containers:
        - name: datacollection
          image: jangdaniel/getrand-datacollection-service:v1.0.5
          ports:
            - containerPort: 5003
```

```
apiVersion: v1
kind: Service
metadata:
  name: datacollectionservice
  namespace: getrand
  labels:
    service: datacollectionservice
spec:
  selector:
    service: datacollectionservice
  ports:
    - port: 5003
      targetPort: 5003
  type: ClusterIP
```

### 🌐 Ingress 설정 (`ingress-setting.yaml`)
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-setting
  namespace: getrand
spec:
  ingressClassName: nginx
  rules:
    - http:
        paths:
          - backend:
              service:
                name: mimi-user
                port:
                  number: 5678
            path: /rental
            pathType: Prefix
```

---

## ⚙️ Jenkins CI/CD 파이프라인 (`Jenkinsfile`)

```groovy
pipeline{
    agent any

    environment{
        APP_REPO_URL = 'https://github.com/CoderJDan/getRand_datacollectionservice.git'
        GITHUB_CREDENTIAL_ID = 'github_hook_id'
        DOCKERHUB_CREDENTIAL_ID = 'docker-hub-access'
        DOCKERHUB_REPOSITORY_IMAGE = 'jangdaniel/getrand-datacollection-service'
        DOCKERHUB_TAG = "v1.0.${env.BUILD_NUMBER}"
    }
    stages{
        stage("git clone"){
            steps{
                git branch: 'develop', credentialsId: 'github_hook_id', url: 'https://github.com/CoderJDan/getRand_datacollectionservice.git'
            }
        }
        stage("docker build"){
            steps{
                sh 'docker build -t $DOCKERHUB_REPOSITORY_IMAGE:$DOCKERHUB_TAG .'
            }
        }
        stage("docker push"){
            steps{
                script{
                    docker.withRegistry('https://index.docker.io/v1/', DOCKERHUB_CREDENTIAL_ID){
                        def myImage = docker.image("$DOCKERHUB_REPOSITORY_IMAGE:$DOCKERHUB_TAG")
                        myImage.push()
                    }
                }
            }
        }
    }
}
```

---
## 🔄ArgoCD : 배포 자동화 
![argocd](https://github.com/CoderJDan/getRand-build-release/blob/master/build_release_screenshot/argocd.png?raw=true)


---

## 🔄 GitHub Actions: YAML 파일 동기화

### 📜 `push_yaml_to_repo.yml`
```yaml
name: Push YAML to Another Repo

on:
  push:
    branches:
      - Devops
    paths:
      - "argo/**/*.yaml"
  workflow_dispatch:

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - name: 저장소 A 체크아웃
        uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Git 설정
        run: |
          git config --global user.name "github-actions"
          git config --global user.email "github-actions@github.com"

      - name: 저장소 B (`mimiyaml.git`) 클론
        run: |
          git clone https://x-access-token:${{ secrets.GH_PAT }}@github.com/05Daul/mimiyaml.git repo_b
          cd repo_b
          git checkout daul || git checkout -b daul
          git pull origin daul --rebase

      - name: YAML 파일 복사 및 푸시
        run: |
          mkdir -p repo_b/argo
          cp -r argo/*.yaml repo_b/argo/ || echo "No YAML files to copy"
          cd repo_b
          git add .
          if ! git diff --cached --exit-code; then
            git commit -m "자동 업데이트: 저장소 A에서 YAML 파일 변경됨"
            git push origin daul || (sleep 5 && git push origin daul)
          else
            echo "No changes detected, skipping push."
          fi
```

---

---
## 💻 전체 흐름도

![process](https://github.com/CoderJDan/getRand-build-release/blob/master/build_release_screenshot/process.png?raw=true)

---
---

## ✅ 실행 및 검증 방법
```sh
# Kubernetes 배포
kubectl apply -f getrand-datacollection-deploy.yaml
kubectl apply -f getrand-datacollection-service.yaml
kubectl apply -f ingress-setting.yaml

# 배포 확인
kubectl get pods -n getrand
kubectl get svc -n getrand
```



## 🎯 프로젝트 정보
📌 **Data Collection 레포지토리:** [getRand_datacollectionservice](https://github.com/CoderJDan/getRand_datacollectionservice.git)
📌 **User 레포지토리:** [getRand_userservice](https://github.com/CoderJDan/getRand_userservice.git)
📌 **Analystic 레포지토리:** [getRand_analysticservice](https://github.com/CoderJDan/getRand_analysticservice.git)

