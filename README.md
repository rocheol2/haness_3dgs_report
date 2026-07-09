# 3DGS 측량 정확도 검증 논문 작성 하네스

3D Gaussian Splatting(3DGS)의 측량학적 현장 적용성을 검증하는 학술 논문을 연구설계→실험→분석→집필→투고준비→통합검증까지 Claude Code 에이전트 팀이 협업하여 생성하는 하네스입니다.

이 하네스는 [revfactory/harness-100](https://github.com/revfactory/harness-100)의 `ko/98-academic-paper` 예제를 3DGS 측량 정확도 검증 연구(스마트폰 영상 촬영, 국내 학술지 투고 대상)에 맞게 커스터마이징한 것입니다.

## 구조

```
.claude/
├── agents/
│   ├── research-designer.md    — 연구 설계자
│   ├── experiment-manager.md   — 실험 관리자 (촬영 프로토콜, 기준측량 계획)
│   ├── statistical-analyst.md  — 정확도 분석가
│   ├── paper-writer.md         — 논문 작성자
│   ├── submission-preparer.md  — 투고 준비자
│   └── executive-summarizer.md — 통합 검증(QA)
├── skills/
│   ├── academic-paper/         — 오케스트레이터
│   ├── research-methodology/   — 정확도 검증 방법론 가이드
│   └── citation-standards/     — 인용/참고문헌 표준
└── CLAUDE.md
```

## 사용법

이 저장소를 클론한 폴더를 프로젝트 루트로 하여 Claude Code 세션을 시작한 뒤, `/academic-paper`를 트리거하거나 "3DGS 정확도 검증 논문 써줘" 같은 자연어로 요청합니다.

자세한 내용은 [.claude/CLAUDE.md](.claude/CLAUDE.md)를 참고하세요.
