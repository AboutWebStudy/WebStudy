### 👉 파이프라인이란?

---

- 소프트웨어를 개발, 빌드, 테스트, 배포하는 일련의 단계를 체계적으로 연결한 워크플로우를 의미
- 각 단계가 순차적으로 진행되며, 자동화를 통해 효율성과 품질을 높임
- 일반적으로 파이프라인은 CI/CD(Continuous Integration/Continuous Deployment)와 관련된 개념으로 많이 사용됨
- 사용하는 이유
    - 자동화를 통해 수작업을 최소화하여 실수를 줄이고 일관성 유지
    - 개발부터 배포까지의 시간을 단축시켜 빠른 피드백 루프를 제공해 효율적
    - 자동화된 테스트를 통해 배포 전 애플리케이션 품질을 확인할 수 있음
    - 여러 팀원이 동시에 작업해도 충돌을 최소화하며, 코드 변경 사항이 항상 통합된 상태를 유지
    <br>
    <br>

### 👉 파이프라인의 주요 요소 및 흐름

---

- 소스 코드(Source Code)
    - 개발자가 코드를 작성하면, 소스 코드가 버전 관리 시스템(Git 등)에 저장됨
- 빌드(Build)
    - 코드를 컴파일하거나, 애플리케이션을 실행 가능한 상태로 패키징
    - 의존성 설치, 코드 병합, 린팅(linting) 등이 포함될 수 있음
- 테스트(Test)
    - 코드가 정상적으로 동작하는지 확인하는 단계
    - 단위 테스트(Unit Test), 통합 테스트(Integration Test), UI 테스트 등이 수행됨
- 배포 준비
    - 실제 배포 전에 애플리케이션을 스테이징 환경에서 테스트하며, 최종 검토 단계를 거침
- 배포
    - 애플리케이션이 최종 사용자 환경(프로덕션)으로 배포
    - 무중단 배포를 활용할 수 있음
- 모니터링 및 피드백
    - 배포 후 애플리케이션 상태를 모니터링하며, 오류를 추적하고 개선 사항을 반영
    <br>
    <br>

### 👉 CI/CD와의 연관성

---

- CI(Continuous Integration)
    - 코드가 변경될 때마다 빌드와 테스트를 자동으로 실행해, 통합 문제를 조기에 발견
- CD(Continuous Deployment)
    - 테스트를 통과한 코드를 자동으로 배포하여 사용자에게 빠르게 전달
    <br>
    <br>

### 👉 파이프라인을 사용하는 도구와 기술

---

- CI/CD 도구 : Jenkins, GitHub Actions, GitLab CI/CD, CircleCI, Travis CI 등
- 배포 도구 : Docker, Kubernetes, Terraform 등
- 모니터링 도구: Prometheus, Grafana, New Relic 등
<br>
<br>

### 👉 파이프라인 흐름

---

- 개발자가 Git에 코드를 커밋 → 푸쉬
- CI 도구가 코드를 가져와 컴파일 및 패키징하며 자동으로 빌드 진행
- 테스트 스크립트 자동으로 실행
- 테스트 성공 시 스테이징 혹은 프로덕션 환경으로 배포
- 모니터링으로 배포 이후 상태를 확인하고 로그를 분석
<br>
<br>

### 👉 Github Actions 파이프라인 예제

---

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline with Monitoring

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 16

      - name: Install Dependencies
        run: npm install

      - name: Run Tests
        run: npm test

      - name: Build Project
        run: npm run build

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to Production
        run: |
          echo "Deploying to production..."
          scp -r ./dist user@production-server:/var/www/html

      - name: Notify via Slack
        uses: slackapi/slack-github-action@v1.23.0
        with:
          slack-bot-token: ${{ secrets.SLACK_BOT_TOKEN }}
          channel-id: 'C12345678'
          text: "Deployment to production is complete!"

  monitor:
    needs: deploy
    runs-on: ubuntu-latest
    steps:
      - name: Monitor Application Health
        run: |
          curl -f http://production-server/health || exit 1
          echo "Health check passed!"

      - name: Check Metrics with Prometheus
        run: |
          curl -f http://prometheus-server/api/v1/query?query=up{job="my-app"} || exit 1
          echo "Prometheus metrics are OK!"

```

- on : 트리거를 정의 (push, pull_request 등)
- jobs : 실행할 작업들을 정의
- steps : 각 작업 안에서 실행할 단계를 정의
- runs-on : 파이프라인이 실행되는 환경 (ubuntu-latest, windows-latest 등)
<br>
<br>

### 👉 Jenkins 파이프라인 예제

---

- Jenkins에서는 Declarative Pipeline(선언형 파이프라인)과 Scripted Pipeline(스크립트형 파이프라인) 두 가지 문법이 있는데, 예제는 Declarative Pipeline

```groovy
pipeline {
    agent any

    environment {
        GRAFANA_API_KEY = credentials('grafana-api-key')
        SLACK_CHANNEL = '#deployment'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                sh 'scp target/my-app.jar user@production-server:/path/to/deploy'
            }
        }

        stage('Monitor Application') {
            steps {
                script {
                    def healthCheck = sh(script: "curl -f http://production-server/health || exit 1", returnStatus: true)
                    if (healthCheck != 0) {
                        error "Health check failed!"
                    }
                }
            }
        }

        stage('Check Grafana Dashboard') {
            steps {
                script {
                    def grafanaResponse = sh(script: """
                      curl -H "Authorization: Bearer ${GRAFANA_API_KEY}" \
                      http://grafana-server/api/dashboards/uid/my-dashboard || exit 1
                    """, returnStatus: true)
                    if (grafanaResponse != 0) {
                        error "Grafana monitoring failed!"
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution completed!'
        }
        success {
            slackSend channel: SLACK_CHANNEL, message: 'Deployment and monitoring successful!'
        }
        failure {
            slackSend channel: SLACK_CHANNEL, message: 'Pipeline failed. Check Jenkins logs.'
        }
    }
}

```

- pipeline : 전체 파이프라인 정의
- agent : 작업 실행 환경 (any, docker 등)
- stages : 작업 단계를 정의
- steps : 각 단계에서 실행할 명령어
- post : 빌드 완료 후 실행할 작업 (always, success, failure)
<br>
<br>

### 👉 GitLab CI/CD 파이프라인 예제

---

```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - deploy
  - monitor

variables:
  NODE_ENV: production
  PROMETHEUS_QUERY: 'up{job="my-app"}'
  PROMETHEUS_URL: 'http://prometheus-server/api/v1/query'
  SLACK_WEBHOOK: $SLACK_WEBHOOK_URL # Slack Webhook은 GitLab Secret에 저장

# Build Stage
build:
  stage: build
  script:
    - echo "Installing dependencies..."
    - npm install
    - echo "Building application..."
    - npm run build
  artifacts:
    paths:
      - dist/

# Test Stage
test:
  stage: test
  script:
    - echo "Running tests..."
    - npm test

# Deploy Stage
deploy:
  stage: deploy
  only:
    - main
  script:
    - echo "Deploying application to production..."
    - scp -r dist/ user@production-server:/var/www/html
  environment:
    name: production
    url: http://production-server

# Monitor Stage
monitor_health:
  stage: monitor
  script:
    - echo "Checking application health..."
    - curl -f http://production-server/health || exit 1
    - echo "Health check passed!"

monitor_metrics:
  stage: monitor
  script:
    - echo "Checking Prometheus metrics..."
    - response=$(curl -s "${PROMETHEUS_URL}?query=${PROMETHEUS_QUERY}")
    - echo "Response: $response"
    - echo "$response" | grep '"status":"success"' || exit 1
    - echo "Prometheus metrics are healthy!"

notify_slack:
  stage: monitor
  script:
    - echo "Sending deployment and monitoring status to Slack..."
    - curl -X POST -H 'Content-type: application/json' --data '{
        "text": "Deployment and monitoring completed successfully!"
      }' $SLACK_WEBHOOK
```

- stages : 파이프라인의 작업 단계 정의
- variables : 파이프라인 전체에서 사용할 변수 정의
- script : 각 단계에서 실행할 명령어
- artifacts : 빌드 결과물을 저장
- only : 특정 조건에서만 실행 (main 브랜치 등)
<br>
<br>

### 👉 도구별 파이프라인 정리

---

| 도구 | 파일명 | 주요 트리거 | 환경 설정 | 주로 사용하는 언어 |
| --- | --- | --- | --- | --- |
| Github Actions | .github/workflows/*.yml | push, pull_request | runs-on | YAML |
| Jenkins | Jenkinsfile | Git 이벤트, 스케줄링 | agent | Groovy |
| GitLab CI/CD | .gitlab-ci.yml | Git 이벤트 | stages | YAML |
