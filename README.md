# SMART on FHIR 환자 요약 + 약물-알레르기 CDS (합성데이터 E2E)

본 프로젝트는 **헬스케어 상호운용성(FHIR) + 인증/인가(OAuth2/SMART 스타일) + 접근통제(RBAC) + 감사로그(Audit)** 패턴을 개인 프로젝트로 재현한 **연구/데모 목적**의 시스템입니다.  
**의료기기(진단/치료 목적)가 아니며, 실제 임상 사용을 의도하지 않습니다.**  
데이터는 **합성(가짜) EHR 데이터**만 사용합니다. (자세한 내용은 DISCLAIMER.md 참고)

**Notion**: https://www.notion.so/SMART-on-FHIR-CDS-2d7c2fb07df980e8ad46dd50aecfe8a6

**전자연구노트(ELN)**: https://mynotebook.labarchives.com/MTU3NTA4Ni41fDEyMTE2MDUvMTIxMTYwNS9Ob3RlYm9vay8yNDAzMTIyMzM2fDM5OTgyOTYuNQ==/page/11410843-2

---

## 0) 프로젝트 개요
이 프로젝트는 다음을 제공합니다.

- OAuth2 기반(Dev 모드) 로그인/토큰 흐름 및 스코프 기반 접근 제어(“SMART 스타일”)
- 로컬 FHIR 서버(HAPI FHIR JPA)에서 환자 데이터 조회
- 임상의 관점의 “환자 요약” 화면
  - 문제(Condition), 약물(Medication*), 알레르기(AllergyIntolerance), 검사/활력징후(Observation) 등
- 규칙 기반 CDS: **약물–알레르기 상호작용 경고**
  - 결정론적(rule-based) 규칙 엔진
  - 경고마다 “왜 떴는지” 근거(리소스/필드) 표시
- **감사로그(Audit Log)**: 누가/언제/어떤 환자 데이터에 접근했고, 어떤 경고가 발생했는지 기록

---

## 1) 데모(영상/스크린샷)
- [ ] 데모 영상(3~5분): 로그인 → 환자 선택 → 요약 조회 → 경고 확인 → 감사로그 확인
- [ ] 스크린샷: 환자 선택 / 요약 / 경고 / 감사로그 / (관리자 화면이 있다면 추가)

---

## 2) 아키텍처
구성 요소(예시)
- FHIR 서버: HAPI FHIR JPA Server Starter
- 데이터: Synthea 생성 합성 환자 데이터(FHIR 번들)
- 앱:
  - Frontend: 환자 선택/요약/경고/감사 화면
  - Backend: FHIR 클라이언트, 정규화(normalization), 규칙엔진, 감사로그 저장
- 인증/인가:
  - OAuth2 스타일 토큰(Dev 모드)
  - 스코프 → 권한 매핑 + RBAC(역할 기반 접근통제)
- 관측/운영:
  - 구조화 로그 + 감사로그 테이블/조회 API

추가 예정 다이어그램
- [ ] 시스템 아키텍처 다이어그램
- [ ] 인증 시퀀스 다이어그램(Discovery → authorize → token → FHIR 호출)
- [ ] 데이터 플로우 다이어그램(FHIR → 정규화 → UI/CDS/Audit)

---

## 3) 기능 목록
### 3.1 환자 요약(Patient Summary)
- Patient: 인구통계/식별정보(최소화)
- Condition: 문제/진단 목록
- MedicationRequest / MedicationStatement: 처방/복용(서버 구성에 따라 다를 수 있음)
- AllergyIntolerance: 알레르기/과민반응
- Observation: 검사/활력징후

### 3.2 CDS: 약물–알레르기 경고(규칙 기반)
- 규칙 엔진: 결정론적(동일 입력 → 동일 결과)
- 심각도/우선순위, 메시지 템플릿
- 오탐 방지(예: inactive, entered-in-error 등 상태값 고려 가능)
- “설명가능성”: 경고 발생 근거를 화면/응답에 포함

### 3.3 보안/권한/감사로그(데모 수준)
- 역할(Role): clinician / nurse / admin (예시)
- 스코프 기반 접근제어 + RBAC 매핑
- 감사로그:
  - 데이터 접근 이벤트(환자 조회, 리소스 조회 등)
  - CDS 이벤트(경고 발생, 경고 확인 등)

---

## 4) 데이터(합성 데이터만 사용)
본 프로젝트는 **Synthea로 생성한 합성 의료기록**만 사용하며, 실제 환자 데이터는 포함하지 않습니다.

- 데이터 생성: ./scripts/generate_synthea.sh
- FHIR 서버 적재: ./scripts/import_fhir_bundles.sh

---

## 5) 실행 방법(로컬/Docker)
### 요구사항
- Docker
- Docker Compose

### 실행
- docker compose up --build

### 중지
- docker compose down

### 엔드포인트(예시)
- App: http://localhost:8080
- FHIR base: http://localhost:8081/fhir
- CapabilityStatement: GET /metadata


---

## 6) 테스트
- 단위 테스트(Unit)
  - 규칙 엔진(rule engine)
  - 스코프/권한 매핑
  - 데이터 정규화 로직
- 통합 테스트(Integration)
  - FHIR 서버 쿼리 정상 동작
  - 토큰 검증
  - 감사로그 기록/조회

---

## 7) 평가(Evaluation) 지표(엔지니어링/연구 공통)
- 데이터 커버리지
  - 알레르기 + 약물 동시 존재 환자 비율
- 알람률(경고 빈도)
  - 환자 조회 100건당 경고 수
  - 환자 1명당 경고 수 분포(경고 남발 여부)
- 설명가능성
  - 경고 근거 필드/리소스 제공률
- 성능/신뢰성
  - 요약 화면 로딩 지연(P50/P95)
  - FHIR 호출 실패율, 재시도/타임아웃 처리

---

## 8) 로드맵
- [ ] MVP: 로컬 FHIR + 환자요약 + 기본 CDS + 감사로그
- [ ] SMART discovery 기반 구성(가능하면)
- [ ] 용어 정규화(예: RxNorm/ATC 등) 옵션 적용
- [ ] 사용성 테스트(비공식) 및 UI 개선
- [ ] 모니터링(에러율/지연) 대시보드 추가

---

## 9) 문서/리포트
- docs/engineering-report.md (설계/보안/테스트/성능/한계)
- docs/short-paper.md (연구 형식 요약 논문 초안)
- docs/threat-model.md (최소 위협모델)
- docs/diagrams/ (아키텍처/시퀀스/데이터플로우 다이어그램)

---

## 10) 라이선스
- MIT

---

## 11) 면책(중요)
- 본 소프트웨어는 **연구/학습/데모 목적**으로 제작되었습니다.
- **의료기기 또는 임상 의사결정 지원 도구로 승인/인증된 제품이 아니며**, 실제 환자 진단/치료/처방에 사용해서는 안 됩니다.
- 본 프로젝트는 **합성 데이터만** 사용합니다.
