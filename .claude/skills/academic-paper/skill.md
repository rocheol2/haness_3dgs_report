---
name: academic-paper
description: "3D Gaussian Splatting(3DGS)의 측량학적 현장 적용성 검증 논문을 연구설계→실험→분석→집필→투고준비→통합검증까지 에이전트 팀이 협업하여 한 번에 생성하는 풀 연구 파이프라인. '3DGS 논문 써줘', '측량 정확도 검증 논문', '현장 적용성 검증 연구', '연구 설계해줘', '정확도 분석해줘', '학술지 투고 준비', '논문 집필', '가설 검증' 등 3DGS 측량 정확도 검증 논문 작성 전반에 이 스킬을 사용한다. 기존 데이터나 초고가 있는 경우에도 분석/리라이팅/투고 준비를 지원한다. 단, 실제 촬영·측량 실행, 3DGS 학습 실행, 통계 소프트웨어 실행, 학회 시스템 로그인 및 업로드는 이 스킬의 범위가 아니다."
---

# Academic Paper — 3DGS 측량 정확도 검증 논문 풀 파이프라인

3DGS의 측량학적 현장 적용성 검증 논문의 연구설계→실험→분석→집필→투고준비→통합검증을 에이전트 팀이 협업하여 한 번에 생성한다. **이 파이프라인의 연구는 3DGS 알고리즘 자체의 개선이 아니라, 기존 3DGS 구현(스마트폰 영상 기반)의 현장 측량 정확도·실무 적용 가능성을 검증하는 것을 목적으로 한다.**

## 실행 모드

**에이전트 팀** — 6명이 SendMessage로 직접 통신하며 교차 검증한다.

## 에이전트 구성

| 에이전트 | 파일 | 역할 | 타입 |
|---------|------|------|------|
| research-designer | `.claude/agents/research-designer.md` | 연구 질문, 가설, 변수 정의, 선행연구 (현장 적용성 검증 프레임) | general-purpose |
| experiment-manager | `.claude/agents/experiment-manager.md` | 스마트폰 촬영 프로토콜, 기준측량(GCP/TS/TLS) 계획, 3DGS 처리 파이프라인 | general-purpose |
| statistical-analyst | `.claude/agents/statistical-analyst.md` | 정확도 평가(RMSE/C2C 등), 코드 생성, 시각화 | general-purpose |
| paper-writer | `.claude/agents/paper-writer.md` | 국내 학술지 형식(국영문 초록) IMRaD 집필, 인용 관리 | general-purpose |
| submission-preparer | `.claude/agents/submission-preparer.md` | 국내 학술지 선정, 포맷팅, 커버레터 | general-purpose |
| executive-summarizer | `.claude/agents/executive-summarizer.md` | 전 산출물 교차 검증(QA), 통합 리뷰 보고서 | general-purpose |

## 워크플로우

### Phase 1: 준비 (오케스트레이터 직접 수행)

1. 사용자 입력에서 추출한다:
    - **연구 초점**: 현장 적용성 검증(기본값) — 알고리즘 개선을 요청하는 경우 범위 밖임을 안내하고 확인
    - **대상 현장**: 현장 유형, 규모, 접근성
    - **입력 데이터 취득 방식**: 스마트폰 촬영 영상(기본값), 필요 시 UAV/지상카메라 병행
    - **기준측량(Ground Truth) 방법**: GNSS-RTK GCP / 토탈스테이션(TS) / 지상라이다(TLS) — 미확정이면 experiment-manager가 비교표로 제안
    - **연구 수준** (선택): 학부/석사/박사 수준
    - **타깃 학술지** (선택): 국내 학술지 기본값, 특정 학술지 지정 가능
    - **기존 자료** (선택): 촬영 영상, 측량 데이터, 분석 결과, 초고
    - **제약 조건** (선택): 마감일, 분량, 특이사항
2. 위 항목 중 연구의 근본 방향을 바꾸는 결정(연구 초점, 기준측량 방법, 촬영 플랫폼, 투고 대상)이 불명확하면 **AskUserQuestion으로 사용자에게 확인한 뒤 진행**한다
3. `_workspace/` 디렉토리를 프로젝트 루트에 생성한다
4. 입력을 정리하여 `_workspace/00_input.md`에 저장한다
5. 기존 파일이 있으면 `_workspace/`에 복사하고 해당 Phase를 건너뛴다
6. 요청 범위에 따라 **실행 모드를 결정**한다 (아래 "작업 규모별 모드" 참조)

### Phase 2: 팀 구성 및 실행

| 순서 | 작업 | 담당 | 의존 | 산출물 |
|------|------|------|------|--------|
| 1 | 연구 설계 | designer | 없음 | `_workspace/01_research_design.md` |
| 2 | 촬영/측량 프로토콜 | experiment | 작업 1 | `_workspace/02_experiment_protocol.md` |
| 3 | 정확도 분석 | analyst | 작업 1, 2 | `_workspace/03_analysis_report.md` |
| 4 | 논문 집필 | writer | 작업 1, 2, 3 | `_workspace/04_manuscript.md` |
| 5 | 투고 준비 | preparer | 작업 1, 4 | `_workspace/05_submission_package.md` |
| 6 | 통합 검증(QA) | summarizer | 작업 1~5 전체 | `_workspace/06_review_report.md` |

**팀원 간 소통 흐름:**
- designer 완료 → experiment에게 변수·대상현장 계획 전달, analyst에게 가설·정확도지표 후보 전달, writer에게 이론적 배경 전달, preparer에게 연구 분야 전달
- experiment 완료 → analyst에게 데이터 구조·좌표계·기준측량 원시데이터 전달, writer에게 연구방법 자료 전달
- analyst 완료 → writer에게 결과 자료(정확도 지표 + Table/Figure) 전달, preparer에게 분석 코드 전달
- writer 완료 → preparer에게 완성 원고 전달
- 모든 산출물 완료 → summarizer가 전체 정합성 교차 검증. 🔴 필수 수정 발견 시 해당 에이전트에게 SendMessage로 수정 요청 → 재작업 → 재검증 (최대 2회)

### Phase 3: 통합 및 최종 산출물

1. `_workspace/06_review_report.md`의 검증 결과를 확인한다
2. 🔴 필수 수정이 모두 반영되었는지 확인한다
3. 최종 요약을 사용자에게 보고한다:
    - 연구 설계 — `01_research_design.md`
    - 실험 프로토콜 — `02_experiment_protocol.md`
    - 정확도 분석 — `03_analysis_report.md`
    - 논문 원고 — `04_manuscript.md`
    - 투고 패키지 — `05_submission_package.md`
    - 통합 리뷰 보고서 — `06_review_report.md`

## 작업 규모별 모드

| 사용자 요청 패턴 | 실행 모드 | 투입 에이전트 |
|----------------|----------|-------------|
| "3DGS 논문 써줘", "정확도 검증 논문 작성" | **풀 파이프라인** | 6명 전원 |
| "연구 설계만 해줘" | **설계 모드** | designer 단독 |
| "촬영/측량 프로토콜만 짜줘" + 연구설계 | **프로토콜 모드** | experiment 단독 |
| "이 데이터 정확도 분석해줘" + 데이터 | **분석 모드** | analyst + writer |
| "논문 초고 다듬어줘" + 초고 | **리라이팅 모드** | writer + preparer |
| "투고 준비해줘" + 원고 | **투고 모드** | preparer 단독 |
| "이 연구 설계로 나머지 진행해줘" + 설계서 | **후속 모드** | experiment → analyst → writer → preparer → summarizer |
| "이 원고 검토해줘" | **검토 모드** | summarizer 단독 |

**기존 파일 활용**: 사용자가 촬영 데이터, 측량 데이터, 초고, 설계서 등을 제공하면, 해당 파일을 `_workspace/`에 복사하고 해당 단계를 건너뛴다.

## 데이터 전달 프로토콜

| 전략 | 방식 | 용도 |
|------|------|------|
| 파일 기반 | `_workspace/` 디렉토리 | 주요 산출물 저장 및 공유 |
| 메시지 기반 | SendMessage | 실시간 핵심 정보 전달, 수정 요청 |

파일명 컨벤션: `{순번}_{산출물}.{확장자}`

## 에러 핸들링

| 에러 유형 | 전략 |
|----------|------|
| 선행연구 검색 실패 | 사용자 제공 참고문헌 중심으로 작업, "제한적 문헌 검토" 명시 |
| 연구 초점이 "알고리즘 개선"으로 흘러감 | 이 파이프라인의 범위(현장 적용성 검증)를 안내하고, 알고리즘 개선이 필요하면 별도 연구 트랙임을 확인 |
| 기준측량 방법 미확정 | experiment-manager가 비교표 제시 → AskUserQuestion으로 사용자 확인 후 진행 |
| 데이터 부재 | 분석 전략과 코드만 제공, "데이터 수집 후 실행" 안내 |
| 에이전트 실패 | 1회 재시도 → 실패 시 해당 산출물 없이 진행, 통합 리뷰 보고서에 누락 명시 |
| 가설-결과 불일치 | 논문작성자가 고찰에서 설명, 원인(좌표정합 오차, 촬영조건 등) 분석 제안 |
| 검증에서 🔴 발견 | 해당 에이전트에 수정 요청 → 재작업 → 재검증 (최대 2회) |

## 테스트 시나리오

### 정상 흐름
**프롬프트**: "스마트폰 영상으로 촬영한 3DGS 결과의 측량 정확도를 검증하는 논문을 써줘. 기준측량은 아직 안 정했어."
**기대 결과**:
- 설계: 현장 적용성 검증 프레임, 정확도 지표를 종속변수로 하는 가설 수립
- experiment-manager가 GCP/TS/TLS 비교표를 제시 → AskUserQuestion으로 사용자 확인
- 분석: 채택된 기준측량에 맞는 RMSE/C2C 등 정확도 지표 산출
- 원고: 국영문 초록 병기, IMRaD 구조, "참여자" 대신 "연구 대상 현장 및 데이터" 섹션
- 투고: 국내 측량·공간정보 학술지 3개 추천
- 검증: summarizer가 기준측량↔분석↔원고 정합성 확인, 정합성 매트릭스 전항목 확인

### 기존 데이터 활용 흐름
**프롬프트**: "이 정확도 분석 결과로 논문 집필이랑 투고 준비해줘" + 분석 결과 파일 첨부
**기대 결과**:
- 연구 설계/실험 단계 건너뜀
- 제공된 분석 결과를 `_workspace/03_analysis_report.md`로 정리
- writer → preparer → summarizer 순서로 진행

### 에러 흐름
**프롬프트**: "3DGS로 정확도 개선하는 새 알고리즘 논문 써줘"
**기대 결과**:
- research-designer가 이 파이프라인의 범위(기존 구현의 현장 적용성 검증)와 사용자 요청(알고리즘 개선) 간 불일치를 인지
- AskUserQuestion으로 "현장 적용성 검증으로 진행할지, 알고리즘 개선 연구가 필요한지" 확인 요청

## 에이전트별 확장 스킬

| 확장 스킬 | 경로 | 대상 에이전트 | 역할 |
|----------|------|-------------|------|
| research-methodology | `.claude/skills/research-methodology/skill.md` | research-designer, statistical-analyst | 정확도 검증 설계, 기준측량 선정, 정확도 표준(ASPRS 등), 통계 분석 |
| citation-standards | `.claude/skills/citation-standards/skill.md` | paper-writer, submission-preparer | 국문/APA/IEEE 인용, IMRaD, 국내 학술지 투고 준비 |
