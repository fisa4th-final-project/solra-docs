# 🗂️ solra-docs

> SOLRA 프로젝트의 모든 기술 문서와 운영 기록을 구조적으로 관리하는 문서 저장소

---

## 📌 개요

`solra-docs`는 SOLRA 시스템 개발 및 운영 전반에 대한 **기술 문서, 정책, 의사결정 기록, 회의록, 인프라 설계서 등**을 통합적으로 관리하는 레포지토리입니다.  
프로젝트에 참여하는 누구나 시스템을 이해하고 협업할 수 있도록 문서 기반 개발 문화를 지향합니다.

---

## 🧾 문서 구조

문서는 기능별/도메인별로 분류되어 있으며, 다음과 같은 카테고리로 구성됩니다:

- `project/`: 프로젝트 개요, 일정, 역할 분담, 회의록, 회고 등 협업 기반 문서
    
- `infra/`: Kubernetes, Argo CD, VMware, HAProxy 등 인프라 운영 문서
    
- `backend/`: 백엔드 설계, API 명세, Redis 연동 방식 등
    
- `database/`: ERD, 테이블 설계, 초기화 SQL, MySQL 설정 등
    
- `monitoring/`: Prometheus, Grafana, Alertmanager 등 모니터링 구성
    
- `cicd/`: GitLab, Jenkins 설정, 배포 파이프라인 구성
    

---

## 🧑‍💻 협업 규칙

- 문서 추가/수정 시 Pull Request를 사용합니다.
    
- 가능한 한 **마크다운 양식**을 통일감 있게 사용합니다.
    
- 기술적 선택에 대한 근거나 대안도 함께 문서화합니다.
    

---

## 🔗 관련 레포지토리

|레포지토리|설명|
|---|---|
|[solra-backend](https://github.com/solra-org/solra-backend)|백엔드 API 서버|
|[solra-frontend](https://github.com/solra-org/solra-frontend)|프론트엔드 웹 애플리케이션|
|[solra](https://github.com/solra-org/solra)|Argo CD 기반 K8s 리소스 배포용 레포|

---

## 📝 목표

> 모든 기술적 지식과 운영 이력을 문서화함으로써  
> **"사람이 바뀌어도 시스템은 흔들리지 않도록"** 하는 것이 `solra-docs`의 궁극적인 목적입니다.

---