# NAIT Project

AIoT 기반 감정 컨디션 모니터링 시스템 **NAIT**의 GitHub Organization입니다.

사용자가 라즈베리파이의 아바타 화면에서 자가진단 질문에 답하면,
MediaPipe 기반 얼굴 반응 분석 결과와 함께 중앙 서버로 전달됩니다.

중앙 서버는 표정 해석 모듈과 GPT를 이용해 맞춤형 추가 질문을 생성하고,
최종 결과를 Central DB와 Mobius oneM2M 플랫폼에 저장합니다.

## System Flow

```text
User
  ↓
Raspberry Pi
  ├─ Touch UI
  └─ MediaPipe Face Analysis
  ↓
Central Server
  ├─ Expression Interpretation
  ├─ AI Question Generation
  ├─ Score Calculation
  ├─ Central DB
  └─ Mobius oneM2M
  ↓
Web / Raspberry Pi Result Screen
```

## Repositories

| Repository | 역할 |
|---|---|
| `NAIT-Device` | Raspberry Pi, 카메라, MediaPipe, 터치형 아바타 UI |
| `NAIT-AI` | 표정 해석, GPT 추가 질문 생성 |
| `NAIT-BE` | 사용자·기기·세션 관리, DB, 점수 계산, Mobius 연동 |
| `NAIT-FE` | 사용자 결과와 최근 기록을 보여주는 웹페이지 |

## Collaboration

모든 기능 개발은 다음 순서로 진행합니다.

```text
Issue 생성
→ 작업 Branch 생성
→ Commit 및 Push
→ Pull Request 생성
→ Review
→ Squash Merge
```

`main` 브랜치에는 직접 Push하지 않습니다.
