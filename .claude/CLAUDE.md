# 3DGS Field Validation Paper Harness

3D Gaussian Splatting(3DGS)의 측량학적 현장 적용성 검증 논문의 연구설계→실험→분석→집필→투고준비→통합검증을 에이전트 팀이 협업하여 생성하는 하네스.

**연구 초점**: 3DGS 알고리즘 자체의 개선이 아니라, 스마트폰 촬영 영상 기반 3DGS 재구성 결과가 실제 측량 현장에서 요구되는 정확도 기준을 충족하는지 정량적으로 검증하는 것.

## 구조

```
.claude/
├── agents/
│   ├── research-designer.md    — 연구 설계자 (연구 질문, 가설, 변수 정의, 선행연구)
│   ├── experiment-manager.md   — 실험 관리자 (스마트폰 촬영 프로토콜, 기준측량 계획, 3DGS 처리 파이프라인)
│   ├── statistical-analyst.md  — 정확도 분석가 (RMSE/C2C 등 정확도 평가, 시각화)
│   ├── paper-writer.md         — 논문 작성자 (국내 학술지 형식, IMRaD 집필)
│   ├── submission-preparer.md  — 투고 준비자 (국내 학술지 선정, 포맷팅)
│   └── executive-summarizer.md — 통합 검증(QA) — 전 산출물 교차 검증, 통합 리뷰 보고서
├── skills/
│   ├── academic-paper/
│   │   └── skill.md            — 오케스트레이터 (팀 조율, 워크플로우, 에러핸들링)
│   ├── research-methodology/
│   │   └── skill.md            — 정확도 검증 방법론 가이드 (기준측량 선정, 정확도 표준, 통계)
│   └── citation-standards/
│       └── skill.md            — 국내 학술지 인용/참고문헌 표준 (해외 저널 대비 스타일 포함)
└── CLAUDE.md                   — 이 파일
```

## 사용법

`/academic-paper` 스킬을 트리거하거나, "3DGS 정확도 검증 논문 써줘" 같은 자연어로 요청한다.

연구의 근본 방향(연구 초점, 기준측량 방법, 촬영 플랫폼, 투고 대상)이 불명확한 경우, 오케스트레이터/에이전트가 AskUserQuestion으로 먼저 확인한 뒤 진행한다.

## 산출물

모든 산출물은 `_workspace/` 디렉토리에 저장된다:
- `00_input.md` — 사용자 입력 정리
- `01_research_design.md` — 연구 설계서
- `02_experiment_protocol.md` — 촬영 및 기준측량 프로토콜
- `03_analysis_report.md` — 정확도 분석 보고서
- `04_manuscript.md` — 논문 원고 (국영문 초록 병기)
- `05_submission_package.md` — 투고 패키지
- `06_review_report.md` — 통합 리뷰 보고서 (executive-summarizer 작성)
