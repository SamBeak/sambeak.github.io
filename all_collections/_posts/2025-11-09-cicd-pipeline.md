---
layout: post
title: 🚀 CI/CD 파이프라인 구축 완벽 가이드 (GitHub Actions, Docker, 배포 전략)
date: 2025-11-09
categories:
  [
    "SamBeak",
    "CI/CD",
    "DevOps",
    "GitHub Actions",
    "Docker",
    "Deployment",
  ]
---

# CI/CD란 무엇인가

<br>
개발자가 코드를 작성하고 배포하는 과정, <br>
매번 수동으로 하면 시간이 오래 걸리고 실수할 수 있다. <br><br>

CI/CD는 이런 과정을 자동화하는 방법이다. <br><br>

마치 공장의 자동화 생산 라인처럼, <br>
코드를 작성하면 자동으로 테스트하고, <br>
문제가 없으면 자동으로 배포된다. <br><br>

**CI (Continuous Integration)**: 지속적 통합 <br>
개발자들이 작성한 코드를 자동으로 통합하고 테스트 <br><br>

**CD (Continuous Deployment/Delivery)**: 지속적 배포 <br>
테스트를 통과한 코드를 자동으로 서버에 배포 <br><br>

> ## 왜 CI/CD를 배워야 할까?

<br>

**이유 1: 시간 절약** <br>
수동 배포 30분 → 자동 배포 5분 <br><br>

**이유 2: 버그 조기 발견** <br>
코드를 푸시하자마자 자동 테스트 실행 <br><br>

**이유 3: 안정적인 배포** <br>
사람의 실수를 줄이고 일관성 유지 <br><br>

**이유 4: 면접 필수** <br>
DevOps, GitHub Actions는 면접 단골 질문 <br><br>

# 기본 개념 요약

<br>

## 🏷️ CI/CD 파이프라인 단계

<br>

```
1. [개발자] → [Git Push]
   ↓
2. [CI 서버] : 코드 빌드
   - 의존성 설치
   - 컴파일
   ↓
3. [CI 서버] : 테스트 실행
   - 단위 테스트
   - 통합 테스트
   - E2E 테스트
   ↓
4. [CI 서버] : 코드 품질 검사
   - 린트(Lint)
   - 코드 커버리지
   - 보안 스캔
   ↓
5. [CD 서버] : Docker 이미지 빌드
   - Dockerfile로 이미지 생성
   - 레지스트리에 푸시
   ↓
6. [CD 서버] : 배포
   - 개발 환경
   - 스테이징 환경
   - 프로덕션 환경
   ↓
7. [모니터링]
   - 헬스 체크
   - 로그 확인
   - 알림
```

<br>

## 🏷️ CI/CD 도구 비교

<br>

| 도구 | 타입 | 가격 | 특징 | 사용 예시 |
|------|------|------|------|-----------|
| **GitHub Actions** | 클라우드 | 무료~유료 | GitHub 통합, 간편 | 오픈소스, 스타트업 |
| **GitLab CI/CD** | 클라우드/온프레미스 | 무료~유료 | 올인원 플랫폼 | 대기업 |
| **Jenkins** | 온프레미스 | 무료 | 강력하고 유연 | 레거시 시스템 |
| **CircleCI** | 클라우드 | 무료~유료 | 빠른 빌드 | SaaS 서비스 |
| **Travis CI** | 클라우드 | 무료~유료 | 오픈소스 친화적 | 오픈소스 프로젝트 |

<br>

## 🏷️ GitHub Actions 핵심 개념

<br>

### 1. Workflow (워크플로우)

<br>
**개념**: 자동화된 프로세스 전체 <br>
`.github/workflows/` 디렉토리에 YAML 파일로 정의 <br><br>

### 2. Event (이벤트)

<br>
**개념**: 워크플로우를 실행시키는 트리거 <br><br>

**주요 이벤트**:
- `push`: 코드 푸시
- `pull_request`: PR 생성/수정
- `schedule`: 주기적 실행 (cron)
- `workflow_dispatch`: 수동 실행

<br>

### 3. Job (작업)

<br>
**개념**: 하나 이상의 Step으로 구성된 작업 단위 <br>
여러 Job은 병렬 또는 순차 실행 가능 <br><br>

### 4. Step (단계)

<br>
**개념**: Job 내의 개별 작업 <br>
명령어 실행 또는 Action 사용 <br><br>

### 5. Action (액션)

<br>
**개념**: 재사용 가능한 작업 모듈 <br>
GitHub Marketplace에서 다운로드 가능 <br><br>

### 6. Runner (러너)

<br>
**개념**: 워크플로우를 실행하는 서버 <br>
GitHub 제공 또는 Self-hosted <br><br>

## 🏷️ 배포 전략

<br>

### 1. Rolling Deployment (롤링 배포)

<br>
**개념**: 서버를 순차적으로 교체 <br><br>

```
[서버1] [서버2] [서버3] [서버4]
  ↓
[새버전] [서버2] [서버3] [서버4]
  ↓
[새버전] [새버전] [서버3] [서버4]
  ↓
[새버전] [새버전] [새버전] [서버4]
  ↓
[새버전] [새버전] [새버전] [새버전]
```

<br>

**장점**: 무중단 배포, 점진적 업데이트 <br>
**단점**: 배포 시간 오래 걸림, 버전 혼재 <br><br>

### 2. Blue-Green Deployment

<br>
**개념**: 두 개의 환경을 교대로 사용 <br><br>

```
[Blue 환경 - 구버전] ← 현재 트래픽
[Green 환경 - 신버전] ← 배포 중

배포 완료 후:
[Blue 환경 - 구버전]
[Green 환경 - 신버전] ← 트래픽 전환

문제 발생 시 즉시 Blue로 롤백
```

<br>

**장점**: 즉시 롤백 가능, 무중단 배포 <br>
**단점**: 2배의 리소스 필요 <br><br>

### 3. Canary Deployment (카나리 배포)

<br>
**개념**: 일부 트래픽만 신버전으로 전환 <br><br>

```
[구버전] ← 90% 트래픽
[신버전] ← 10% 트래픽

문제없으면 점진적으로 증가:
[구버전] ← 50% 트래픽
[신버전] ← 50% 트래픽

최종:
[신버전] ← 100% 트래픽
```

<br>

**장점**: 위험 최소화, 점진적 검증 <br>
**단점**: 복잡한 트래픽 관리 <br><br>

# 실전 예시

<br>

## 🏷️ GitHub Actions 기본 워크플로우

<br>

### 1. 프로젝트 구조

<br>

```
project/
├── .github/
│   └── workflows/
│       ├── ci.yml          # CI 워크플로우
│       ├── deploy-dev.yml  # 개발 환경 배포
│       └── deploy-prod.yml # 프로덕션 배포
├── src/
├── tests/
├── Dockerfile
└── package.json
```

<br>

### 2. CI 워크플로우 (ci.yml)

<br>

```yaml
name: CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        node-version: [18.x, 20.x]
    
    steps:
      # 1. 코드 체크아웃
      - name: Checkout code
        uses: actions/checkout@v4
      
      # 2. Node.js 설정
      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      
      # 3. 의존성 설치
      - name: Install dependencies
        run: npm ci
      
      # 4. 린트 검사
      - name: Run linter
        run: npm run lint
      
      # 5. 단위 테스트
      - name: Run unit tests
        run: npm test
      
      # 6. 코드 커버리지
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/coverage-final.json
          fail_ci_if_error: true
  
  e2e-test:
    runs-on: ubuntu-latest
    needs: test
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20.x
      
      - name: Install dependencies
        run: npm ci
      
      # 7. Playwright 설치
      - name: Install Playwright
        run: npx playwright install --with-deps
      
      # 8. E2E 테스트
      - name: Run E2E tests
        run: npm run test:e2e
      
      # 9. 테스트 실패 시 스크린샷 업로드
      - name: Upload test results
        if: failure()
        uses: actions/upload-artifact@v3
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
  
  security-scan:
    runs-on: ubuntu-latest
    needs: test
    
    steps:
      - uses: actions/checkout@v4
      
      # 10. 보안 취약점 스캔
      - name: Run security audit
        run: npm audit --audit-level=moderate
      
      # 11. SAST (정적 분석)
      - name: Run CodeQL Analysis
        uses: github/codeql-action/init@v2
        with:
          languages: javascript
      
      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v2
```

<br>

## 🏷️ Docker 빌드 및 배포

<br>

### 1. Dockerfile

<br>

```dockerfile
# 멀티 스테이지 빌드
FROM node:20-alpine AS builder

WORKDIR /app

# 의존성 설치
COPY package*.json ./
RUN npm ci --only=production

# 소스 복사 및 빌드
COPY . .
RUN npm run build

# 프로덕션 이미지
FROM node:20-alpine

WORKDIR /app

# 빌드 결과물만 복사
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package.json ./

# 비root 사용자로 실행
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nodejs -u 1001
USER nodejs

EXPOSE 3000

CMD ["node", "dist/main.js"]
```

<br>

### 2. Docker 빌드 워크플로우

<br>

```yaml
name: Build and Push Docker Image

on:
  push:
    branches: [ main ]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      # 1. Docker Buildx 설정
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      # 2. GitHub Container Registry 로그인
      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      # 3. 메타데이터 추출
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=sha,prefix={{branch}}-
            type=semver,pattern={{version}}
      
      # 4. Docker 이미지 빌드 및 푸시
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

<br>

## 🏷️ 배포 워크플로우 (AWS ECS)

<br>

```yaml
name: Deploy to Production

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      # 1. AWS 자격증명 설정
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-northeast-2
      
      # 2. ECR 로그인
      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2
      
      # 3. Docker 이미지 빌드 및 푸시
      - name: Build and push image to ECR
        id: build-image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          ECR_REPOSITORY: my-app
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          echo "image=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" >> $GITHUB_OUTPUT
      
      # 4. ECS 태스크 정의 업데이트
      - name: Fill in the new image ID in the ECS task definition
        id: task-def
        uses: aws-actions/amazon-ecs-render-task-definition@v1
        with:
          task-definition: task-definition.json
          container-name: my-app
          image: ${{ steps.build-image.outputs.image }}
      
      # 5. ECS 서비스 배포
      - name: Deploy to Amazon ECS
        uses: aws-actions/amazon-ecs-deploy-task-definition@v1
        with:
          task-definition: ${{ steps.task-def.outputs.task-definition }}
          service: my-app-service
          cluster: my-app-cluster
          wait-for-service-stability: true
      
      # 6. Slack 알림
      - name: Notify Slack
        if: always()
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: 'Deployment to production: ${{ job.status }}'
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

<br>

## 🏷️ Blue-Green 배포 전략

<br>

```yaml
name: Blue-Green Deployment

on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target environment'
        required: true
        type: choice
        options:
          - blue
          - green

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-northeast-2
      
      # 1. Blue 또는 Green 환경에 배포
      - name: Deploy to ${{ inputs.environment }} environment
        run: |
          echo "Deploying to ${{ inputs.environment }} environment"
          aws ecs update-service \
            --cluster my-cluster \
            --service my-app-${{ inputs.environment }} \
            --force-new-deployment
      
      # 2. 헬스 체크
      - name: Wait for deployment
        run: |
          aws ecs wait services-stable \
            --cluster my-cluster \
            --services my-app-${{ inputs.environment }}
      
      # 3. 스모크 테스트
      - name: Run smoke tests
        run: |
          ENDPOINT="https://${{ inputs.environment }}.example.com"
          curl -f $ENDPOINT/health || exit 1
          
      # 4. 트래픽 전환 (수동 승인 필요)
      - name: Switch traffic
        if: inputs.environment == 'green' && success()
        run: |
          echo "Ready to switch traffic to green"
          # Route53이나 ALB 설정 변경
```

<br>

## 🏷️ 환경별 배포 설정

<br>

```yaml
name: Deploy to Environment

on:
  push:
    branches: [ main, develop ]

jobs:
  determine-environment:
    runs-on: ubuntu-latest
    outputs:
      environment: ${{ steps.set-env.outputs.environment }}
    steps:
      - id: set-env
        run: |
          if [[ "${{ github.ref }}" == "refs/heads/main" ]]; then
            echo "environment=production" >> $GITHUB_OUTPUT
          else
            echo "environment=development" >> $GITHUB_OUTPUT
          fi
  
  deploy:
    needs: determine-environment
    runs-on: ubuntu-latest
    environment: ${{ needs.determine-environment.outputs.environment }}
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Deploy to ${{ needs.determine-environment.outputs.environment }}
        run: |
          echo "Deploying to ${{ needs.determine-environment.outputs.environment }}"
          
          # 환경별 설정 파일 사용
          cp .env.${{ needs.determine-environment.outputs.environment }} .env
          
          # 배포 스크립트 실행
          ./deploy.sh ${{ needs.determine-environment.outputs.environment }}
```

<br>

# 실전 체크리스트

<br>

## ✅ CI 설정

<br>

- [ ] 자동 빌드 설정
- [ ] 단위 테스트 자동 실행
- [ ] 통합 테스트 자동 실행
- [ ] E2E 테스트 자동 실행
- [ ] 코드 린트 검사
- [ ] 코드 커버리지 측정

<br>

## ✅ 보안

<br>

- [ ] 의존성 취약점 스캔
- [ ] SAST (정적 분석)
- [ ] 시크릿 관리 (GitHub Secrets)
- [ ] 컨테이너 이미지 스캔
- [ ] 최소 권한 원칙

<br>

## ✅ CD 설정

<br>

- [ ] Docker 이미지 자동 빌드
- [ ] 레지스트리 푸시
- [ ] 환경별 배포 설정
- [ ] 롤백 전략
- [ ] 헬스 체크

<br>

## ✅ 모니터링

<br>

- [ ] 배포 성공/실패 알림
- [ ] 빌드 시간 모니터링
- [ ] 테스트 결과 리포트
- [ ] 로그 수집
- [ ] 메트릭 대시보드

<br>

## ✅ 배포 전략

<br>

- [ ] 배포 전략 선택 (Rolling/Blue-Green/Canary)
- [ ] 무중단 배포 구현
- [ ] 롤백 절차 문서화
- [ ] 배포 승인 프로세스
- [ ] 배포 일정 관리

<br>

# 요약

<br>
CI/CD는 개발부터 배포까지의 과정을 자동화하는 필수 도구다. <br><br>

**💎 핵심 포인트**:

1. **CI**: 코드 통합 및 테스트 자동화
2. **CD**: 배포 자동화
3. **GitHub Actions**: 간편한 CI/CD 구현
4. **Docker**: 일관된 배포 환경
5. **배포 전략**: Rolling, Blue-Green, Canary
6. **모니터링**: 배포 후 안정성 확인

<br>

**🎯 CI/CD 파이프라인 단계**:

1. **코드 푸시** → 개발자가 코드 커밋
2. **빌드** → 의존성 설치 및 컴파일
3. **테스트** → 단위/통합/E2E 테스트
4. **코드 품질** → 린트, 커버리지, 보안 스캔
5. **이미지 빌드** → Docker 이미지 생성
6. **배포** → 환경별 자동 배포
7. **모니터링** → 헬스 체크 및 알림

<br>

**📌 배포 전략 비교**:

| 전략 | 속도 | 롤백 | 리소스 | 위험도 |
|------|------|------|--------|--------|
| **Rolling** | 느림 | 느림 | 1배 | 중간 |
| **Blue-Green** | 빠름 | 즉시 | 2배 | 낮음 |
| **Canary** | 중간 | 빠름 | 1.1배 | 매우 낮음 |

<br>

**🚀 Best Practices**:

- **작은 단위로 자주 배포**: 위험 최소화
- **자동화 우선**: 수동 작업 최소화
- **테스트는 필수**: 배포 전 충분한 테스트
- **롤백 계획**: 문제 발생 시 즉시 복구
- **모니터링**: 배포 후 지속적인 관찰
- **문서화**: 배포 프로세스 명확히 기록

<br>

CI/CD는 한 번 설정하면 계속 혜택을 받는다. <br>
초기 설정에 시간이 걸리더라도, <br>
장기적으로는 개발 속도와 안정성을 크게 향상시킨다. <br><br>
