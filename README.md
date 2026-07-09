# 3DGS 측량 정확도 검증 논문 작성 하네스

3D Gaussian Splatting(3DGS)의 **측량학적 현장 적용성**을 검증하는 학술 논문을, Claude Code 위에서 동작하는 6인 에이전트 팀이 협업하여 연구설계 → 실험 → 정확도분석 → 집필 → 투고준비 → 통합검증까지 한 번에 생성하는 하네스입니다.

> **이 연구가 다루는 것**: 스마트폰 영상으로 촬영한 3DGS 재구성 결과가 실제 측량 현장(GNSS-RTK/토탈스테이션/TLS 등 기준측량 대비)에서 요구되는 정확도를 충족하는지 정량적으로 검증
> **이 연구가 다루지 않는 것**: 3DGS 알고리즘 자체의 개선(새로운 손실함수, 초기화 기법 등 novelty 제안)

이 저장소는 [revfactory/harness-100](https://github.com/revfactory/harness-100)의 `ko/98-academic-paper` 예제(사회과학 실증연구용 범용 논문 하네스)를 3DGS 측량 정확도 검증 연구에 맞게 전면 재작성한 것입니다.

---

## 1. 아키텍처 — 이 하네스는 어떻게 동작하는가

이 저장소에는 **실행 가능한 코드가 없습니다.** 오직 마크다운 파일 10개(+README)로 멀티에이전트 시스템을 구현했습니다. 핵심은 Claude Code 하네스가 기본 제공하는 4가지 프리미티브를 조합한 것입니다.

| 프리미티브 | 이 하네스에서의 역할 |
|---|---|
| **Skill 라우팅** | `description` 필드의 자연어 트리거 문구로 "언제 이 스킬을 쓸지" 판단 (별도 커맨드 파서 없음) |
| **Subagent (Agent 도구)** | `agents/*.md` 전체가 서브에이전트 호출 시 시스템 프롬프트로 주입되는 "페르소나 템플릿" |
| **파일시스템 (`_workspace/`)** | 에이전트 간 공유 상태를 저장하는 영속적 메시지 버스 (DB 역할) |
| **SendMessage** | 파일 갱신을 기다리지 않는 실시간 직접 통신 (QA 수정 요청 등) |

### 1.1 트리거 — `description`이 곧 라우터

`academic-paper/skill.md`의 YAML frontmatter에 있는 `description`은 사용자 발화와 의미적으로 매칭되는 디스패치 테이블입니다. "3DGS 논문 써줘", "정확도 검증 논문" 같은 문구가 여기 등록되어 있어야 스킬이 자동으로 로드됩니다. `/academic-paper`로 명시 호출도 가능합니다.

### 1.2 오케스트레이터 — `academic-paper/skill.md`는 제어 흐름을 표(table)로 인코딩

이 파일이 사실상 파이프라인의 `main()` 함수입니다.

- **Phase 1(준비)**: 연구 초점·기준측량 방법·촬영 플랫폼·투고 대상 등 근본 방향이 불명확하면 **AskUserQuestion으로 먼저 확인**하도록 명시되어 있습니다. `_workspace/00_input.md`에 정리된 입력을 저장합니다.
- **Phase 2(팀 실행)**: 아래 "의존성 그래프" 표가 곧 오케스트레이터가 따라야 할 실행 순서/스케줄입니다.
- **Phase 3(통합)**: QA 결과를 확인하고 사용자에게 최종 요약을 보고합니다.

### 1.3 서브에이전트 — 페르소나 주입 템플릿

`.claude/agents/`의 각 파일은 프로세스가 아니라, `subagent_type: general-purpose`로 Agent 도구를 호출할 때 전체가 시스템 프롬프트로 주입되는 템플릿입니다. 모든 에이전트 파일이 공통으로 갖는 4개 섹션이 곧 "인터페이스 계약"입니다.

| 섹션 | 역할 |
|---|---|
| 핵심 역할 | 책임 범위 (함수 시그니처에 해당) |
| 산출물 포맷 | 강제된 출력 스키마 — 다음 에이전트가 이 마크다운 구조를 그대로 파싱해서 읽음 |
| 팀 통신 프로토콜 | 누구에게 무엇을 전달하는지 (API 계약) |
| 에러 핸들링 | try/catch에 해당하는 자연어 폴백 규칙 |

### 1.4 QA 루프 — `executive-summarizer`

전 산출물(01~05)을 교차 검증하는 전담 에이전트입니다. 🔴(필수 수정) / 🟡(권장) / 🟢(참고) 3단계로 문제를 분류하고, 🔴 발견 시 SendMessage로 해당 에이전트에게 수정을 요청 → 재작업 → 재검증(최대 2회)하는 루프를 돕니다. 검증 결과는 `_workspace/06_review_report.md`에 저장됩니다.

이 에이전트는 원본 harness-100 예제에는 없던 것으로, 이번 커스터마이징 과정에서 **신규 설계해 추가**했습니다 (원본에는 오케스트레이터가 Phase 3에서 검증을 "직접 수행"한다고만 되어 있어 QA가 형식화되어 있지 않았음).

### 1.5 확장 스킬 — 에이전트별 온디맨드 지식 주입

`research-methodology`, `citation-standards`는 특정 에이전트를 대상 독자로 명시한 "치트시트"입니다. 매 요청마다 컨텍스트에 다 넣지 않고, 트리거 문구가 매칭될 때만 로드되는 스코프드 RAG 패턴입니다.

---

## 2. 디렉토리 구조

```
haness_3dgs_report/
├── README.md                              — 이 파일
└── .claude/
    ├── CLAUDE.md                          — 하네스 개요/구조/산출물 목록 요약본
    ├── agents/                            — 6인 에이전트 팀 (서브에이전트 페르소나 템플릿)
    │   ├── research-designer.md           — 연구 설계자
    │   ├── experiment-manager.md          — 실험 관리자
    │   ├── statistical-analyst.md         — 정확도 분석가
    │   ├── paper-writer.md                — 논문 작성자
    │   ├── submission-preparer.md         — 투고 준비자
    │   └── executive-summarizer.md        — 통합 검증(QA) — 신규 추가
    └── skills/
        ├── academic-paper/
        │   └── skill.md                   — 오케스트레이터 (트리거, 워크플로우, 에러핸들링, 테스트 시나리오)
        ├── research-methodology/
        │   └── skill.md                   — 정확도 검증 방법론 확장 스킬 (designer·analyst 대상)
        └── citation-standards/
            └── skill.md                   — 인용/참고문헌 표준 확장 스킬 (writer·preparer 대상)
```

(실제로 하네스를 실행하면 프로젝트 루트에 `_workspace/` 디렉토리가 새로 생기고, 아래 "산출물" 표의 파일들이 순서대로 쌓입니다. 초기 저장소에는 포함되어 있지 않습니다.)

---

## 3. 파이프라인 워크플로우

### 3.1 에이전트 구성

| 에이전트 | 파일 | 역할 |
|---|---|---|
| research-designer | `agents/research-designer.md` | 연구 질문, 가설, 독립·종속·통제변수 정의, 선행연구(3DGS/SfM-MVS/사진측량 정확도) 분석 |
| experiment-manager | `agents/experiment-manager.md` | 스마트폰 촬영 프로토콜, 기준측량(GNSS-RTK GCP/TS/TLS) 계획, 3DGS 처리 파이프라인(프레임 추출→SfM→학습→좌표정합) 설계 |
| statistical-analyst | `agents/statistical-analyst.md` | RMSE·MAE·C2C·M3C2 등 정확도 지표 산출 전략, 분석 코드(Python/CloudCompare) 생성, 결과 해석·시각화 |
| paper-writer | `agents/paper-writer.md` | 국내 학술지 관행(국문 본문 + 국영문 초록)에 맞춘 IMRaD 구조 집필 |
| submission-preparer | `agents/submission-preparer.md` | 국내 측량·공간정보 학술지 선정(KCI 등재 여부 등), 포맷팅, 커버레터 |
| executive-summarizer | `agents/executive-summarizer.md` | 전 산출물 교차 검증(QA), 통합 리뷰 보고서 |

### 3.2 의존성 그래프 (Phase 2)

```
1. research-designer      (의존: 없음)
2. experiment-manager     (의존: 1)
3. statistical-analyst    (의존: 1, 2)
4. paper-writer           (의존: 1, 2, 3)
5. submission-preparer    (의존: 1, 4)
6. executive-summarizer   (의존: 1~5 전체, QA)
```

완전 순차 체인입니다 (병렬 실행 구간 없음). 각 단계 완료 시 다음 에이전트에게 필요한 정보를 SendMessage로 직접 전달하기도 합니다 (예: designer 완료 → experiment에게 변수·현장계획, analyst에게 가설·지표후보, writer에게 이론적 배경, preparer에게 연구분야를 각각 전달).

### 3.3 작업 규모별 실행 모드

요청 패턴에 따라 투입되는 에이전트 수를 다르게 가져갑니다.

| 요청 패턴 | 모드 | 투입 에이전트 |
|---|---|---|
| "3DGS 논문 써줘" | 풀 파이프라인 | 6명 전원 |
| "연구 설계만 해줘" | 설계 모드 | designer 단독 |
| "촬영/측량 프로토콜만 짜줘" | 프로토콜 모드 | experiment 단독 |
| "이 데이터 정확도 분석해줘" | 분석 모드 | analyst + writer |
| "논문 초고 다듬어줘" | 리라이팅 모드 | writer + preparer |
| "투고 준비해줘" | 투고 모드 | preparer 단독 |
| "이 연구 설계로 나머지 진행해줘" | 후속 모드 | experiment → analyst → writer → preparer → summarizer |
| "이 원고 검토해줘" | 검토 모드 | summarizer 단독 |

기존에 촬영 영상, 측량 데이터, 초고 등이 있으면 해당 단계를 건너뛰고 `_workspace/`에 바로 반영합니다.

---

## 4. 산출물 (`_workspace/`)

파이프라인을 실행하면 아래 순서로 파일이 생성됩니다.

| 파일 | 담당 에이전트 | 내용 |
|---|---|---|
| `00_input.md` | 오케스트레이터 | 사용자 입력 정리 (연구 초점, 기준측량 방법, 촬영 플랫폼, 투고 대상 등) |
| `01_research_design.md` | research-designer | 연구 질문, 가설, 변수 정의, 선행연구, 연구 공백 |
| `02_experiment_protocol.md` | experiment-manager | 촬영 절차, 기준측량 방법 비교/채택, 3DGS 처리 파이프라인, 파일럿 테스트 계획 |
| `03_analysis_report.md` | statistical-analyst | 정확도 평가 결과(RMSE/C2C 등), 가설 검정, 분석 코드, Table/Figure 설계 |
| `04_manuscript.md` | paper-writer | 논문 원고 (국문초록/Abstract 병기, IMRaD 전체 구조) |
| `05_submission_package.md` | submission-preparer | 타깃 학술지 추천, 커버레터, 투고 체크리스트 |
| `06_review_report.md` | executive-summarizer | 정합성 매트릭스, 🔴🟡🟢 발견 사항, 통합 리뷰 |

---

## 5. 사용법

### 5.1 활성화

이 저장소를 클론한 폴더 자체가 **프로젝트 루트**가 되어야 `.claude/`가 인식됩니다.

```bash
git clone https://github.com/rocheol2/haness_3dgs_report.git
cd haness_3dgs_report
claude   # 또는 VSCode에서 이 폴더를 열고 Claude Code 확장 실행
```

### 5.2 트리거

```
/academic-paper
```
또는 자연어로:
```
3DGS 정확도 검증 논문 써줘
스마트폰 영상으로 촬영한 3DGS 결과의 측량 정확도를 검증하는 논문을 써줘
```

### 5.3 실행 흐름

1. **Phase 1**: 연구 방향(연구 초점/기준측량 방법/촬영 플랫폼/투고 대상)이 불명확하면 AskUserQuestion으로 먼저 확인합니다. 기존 자료(촬영 영상, 측량 데이터, 초고 등)가 있다면 이 시점에 알려주세요.
2. **Phase 2**: 6개 에이전트가 순서대로 실행되며 `_workspace/`에 산출물이 쌓입니다. 중간에 "experiment-manager가 짠 프로토콜에서 기준측량은 TLS로 확정해줘" 같은 수정 지시를 줄 수 있습니다.
3. **Phase 3**: `06_review_report.md`의 QA 결과를 확인하고, 🔴 항목이 있으면 제출 전 반드시 처리합니다.

### 5.4 한계

- 이 하네스는 **글쓰기·설계·분석 전략 수립까지만** 수행합니다. 실제 3DGS 학습, COLMAP 실행, 정확도 계산 코드 실행, GNSS/TS/TLS 현장 측량은 하네스 밖에서 사용자가 직접 수행해야 합니다.
- `statistical-analyst`가 생성하는 Python/CloudCompare 코드는 **작성만** 하며 실행하지 않습니다.
- 선행연구·학술지 조사는 WebSearch 도구에 의존하므로, 세션에 웹 검색이 활성화되어 있어야 합니다.

---

## 6. 원본 대비 커스터마이징 내역

[revfactory/harness-100 `ko/98-academic-paper`](https://github.com/revfactory/harness-100/tree/main/ko/98-academic-paper)(사회과학 실증연구용 범용 논문 하네스) 대비 변경 사항입니다.

| 항목 | 원본 | 이 저장소 |
|---|---|---|
| 연구 프레임 | 가설검정형 실증연구(RCT/서베이), 참여자/설문/IRB 중심 | 현장 실증 검증(Field Validation), 촬영 현장·기준측량 중심, IRB 해당 없음 |
| 정확도/통계 지표 | t-test, ANOVA, 회귀분석 (심리학 관행) | RMSE, MAE, C2C, M3C2, 좌표정합 잔차 (측량·사진측량 관행) |
| 데이터 수집 | 설문지, 척도(Cronbach's α) | 스마트폰 영상 촬영 프로토콜, GNSS-RTK/TS/TLS 기준측량 |
| 논문 형식 | 영문 IMRaD, APA 7th | 국문 본문 + 국영문 초록 병기, "참여자" 대신 "연구 대상 현장 및 데이터" |
| 투고처 | 해외 SSCI 저널, IF·APC 중심 | 국내 측량·공간정보 학술지, KCI 등재 여부 중심 |
| QA 체계 | 오케스트레이터가 Phase 3에서 직접 검증 (전담 에이전트 없음) | `executive-summarizer` 전담 에이전트 신설 — 🔴🟡🟢 심각도 분류 + 재검증 루프 |
| 확장 스킬 | 사회과학 연구방법론, APA/IEEE/Chicago 인용 | 3DGS 처리 파이프라인·정확도 표준(ASPRS 등) 방법론, 국문 참고문헌 양식 우선 |

---

## 7. 출처

- 기반 하네스: [revfactory/harness-100](https://github.com/revfactory/harness-100) (`ko/98-academic-paper`)
- 실행 환경: [Claude Code](https://claude.com/claude-code)
