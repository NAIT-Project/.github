# NAIT 팀 GitHub 사용 가이드

이 문서는 NAIT 프로젝트 팀원이 GitHub Organization, Repository, Issue, Branch, Commit, Pull Request, Project를 어떻게 사용해야 하는지 정리한 안내 문서입니다.

---

## 1. 프로젝트 구조

NAIT 프로젝트는 역할별로 Repository를 분리해서 관리합니다.

| Repository | 역할 |
|---|---|
| `NAIT-Device` | Raspberry Pi, 카메라, MediaPipe, 터치형 아바타 UI |
| `NAIT-AI` | 표정 해석, GPT 추가 질문 생성 |
| `NAIT-BE` | 중앙 서버, 사용자·기기·세션 관리, DB, 점수 계산, Mobius 연동 |
| `NAIT-FE` | 사용자 결과 조회 웹페이지, 대시보드, 그래프 |

전체 흐름은 다음과 같습니다.

```text
User
  ↓
NAIT-Device
  ↓
NAIT-BE
  ├─ NAIT-AI
  ├─ Central DB
  └─ Mobius
  ↓
NAIT-FE / Device Result Screen
```

---

## 2. 가장 중요한 규칙

초기 세팅 이후에는 `main` 브랜치에 직접 Push하지 않습니다.

모든 작업은 아래 순서로 진행합니다.

```text
Issue 생성
→ 작업 Branch 생성
→ 코드 작성
→ Commit
→ Push
→ Pull Request 생성
→ Review
→ Squash Merge
```

즉, 직접 `main`에 올리지 않고 반드시 Issue와 Pull Request를 거쳐 작업합니다.

---

## 3. 처음 참여할 때 해야 할 일

### 3.1 GitHub 초대 수락

팀장이 GitHub Organization 초대를 보내면 이메일 또는 GitHub 알림에서 초대를 수락합니다.

초대를 수락해야 Repository에 접근하고 Issue, Branch, Pull Request를 만들 수 있습니다.

### 3.2 로컬 작업 폴더 만들기

PC에 NAIT 작업 폴더를 하나 만듭니다.

Windows PowerShell 예시:

```powershell
cd $HOME\Desktop
mkdir NAIT
cd NAIT
```

### 3.3 Repository Clone

아래 4개 Repository를 같은 폴더 안에 Clone합니다.

```powershell
git clone https://github.com/조직이름/NAIT-Device.git
git clone https://github.com/조직이름/NAIT-AI.git
git clone https://github.com/조직이름/NAIT-BE.git
git clone https://github.com/조직이름/NAIT-FE.git
```

로컬 폴더 구조는 다음처럼 됩니다.

```text
NAIT/
├── NAIT-Device/
├── NAIT-AI/
├── NAIT-BE/
└── NAIT-FE/
```

각 Repository는 독립된 Git 저장소입니다. 작업할 때는 반드시 해당 Repository 폴더 안으로 들어가서 Git 명령어를 실행합니다.

```powershell
cd NAIT-Device
git status
```

---

## 4. Issue 작성 방법

작업을 시작하기 전에 먼저 Issue를 생성합니다.

### 4.1 Issue 제목 형식

```text
[FEAT] 새로운 기능
[FIX] 버그 수정
[DOCS] 문서 수정
[CHORE] 설정 및 기타 작업
[REFACTOR] 코드 구조 개선
[TEST] 테스트 코드
[CI] CI/CD 설정
[DESIGN] 화면 설계
```

좋은 예시:

```text
[FEAT] MediaPipe 얼굴 지표 추출 기능 구현
[FIX] 카메라 연결 실패 시 예외 처리
[DOCS] Device 실행 방법 문서 작성
[CHORE] 프로젝트 초기 폴더 구조 생성
[DESIGN] 결과 화면 와이어프레임 작성
```

좋지 않은 예시:

```text
작업
수정
AI 만들기
백엔드 완성
```

Issue 하나는 가능한 한 Pull Request 하나로 끝낼 수 있는 크기로 작성합니다.

### 4.2 Issue 본문에 들어갈 내용

```markdown
## 작업 목적

이 작업이 필요한 이유를 작성합니다.

## 작업 항목

- [ ] 세부 작업 1
- [ ] 세부 작업 2
- [ ] 테스트 또는 확인
- [ ] 문서 수정

## 완료 조건

어떤 상태가 되면 작업이 끝났다고 볼 수 있는지 작성합니다.

## 참고 자료

관련 문서, API, 화면, 이전 Issue 등을 작성합니다.
```

---

## 5. GitHub Project 사용법

Project는 전체 작업 현황을 보는 칸반 보드입니다.

### 5.1 Status 의미

| Status | 의미 |
|---|---|
| `Backlog` | 해야 할 작업 후보. 아직 담당자나 일정이 확정되지 않은 상태 |
| `Ready` | 작업 내용과 완료 조건이 정리되어 바로 시작할 수 있는 상태 |
| `In Progress` | 담당자가 실제로 작업 중인 상태 |
| `Review` | Pull Request가 올라왔거나 팀원 확인이 필요한 상태 |
| `Blocked` | 다른 작업, API, 장비, 팀원 확인 때문에 진행이 막힌 상태 |
| `Done` | Pull Request가 Merge되었고 작업이 완료된 상태 |

### 5.2 Status 이동 기준

```text
Backlog
  ↓ 작업 내용 정리 완료
Ready
  ↓ 담당자가 작업 시작
In Progress
  ↓ PR 생성
Review
  ↓ PR Merge
Done
```

진행이 막힌 경우에는 `Blocked`로 옮기고, Issue 댓글에 막힌 이유를 남깁니다.

예시:

```text
Blocked 이유: Backend API 응답 형식이 아직 확정되지 않음
Blocked 이유: Raspberry Pi 카메라 연결 테스트 필요
Blocked 이유: OpenAI API Key 발급 대기
```

---

## 6. Label과 Field 사용법

### 6.1 Issue Label

Issue Label은 작업 종류를 나타냅니다.

| Label | 설명 |
|---|---|
| `feature` | 새로운 기능 구현 |
| `bug` | 오류 또는 예상과 다른 동작 수정 |
| `docs` | README, API 명세, 사용법 등 문서 작업 |
| `chore` | 프로젝트 설정, 폴더 구조, 패키지 등 기타 작업 |
| `refactor` | 기능 변화 없이 코드 구조를 개선하는 작업 |
| `test` | 테스트 코드 추가 또는 수정 |
| `ci` | GitHub Actions, CI/CD, 자동화 설정 |
| `design` | UI/UX, 화면 설계, 와이어프레임 작업 |

### 6.2 Project Area

Area는 작업 영역을 나타냅니다.

| Area | 설명 |
|---|---|
| `Device` | Raspberry Pi, 카메라, MediaPipe, 터치 UI |
| `AI` | 표정 해석, GPT 질문 생성, 프롬프트 |
| `Backend` | 중앙 서버 API, 세션, 점수 계산 |
| `Frontend` | 웹 화면, 대시보드, 그래프 |
| `Database` | DB 테이블, 마이그레이션, 저장 구조 |
| `Mobius` | Mobius oneM2M AE/CNT/CIN 저장 |
| `Common` | 여러 Repository에 공통으로 적용되는 작업 |

### 6.3 Priority

| Priority | 설명 |
|---|---|
| `P0` | 전체 진행을 막는 긴급 작업 |
| `P1` | 현재 마일스톤에서 반드시 해야 하는 작업 |
| `P2` | 중요하지만 우선순위가 중간인 작업 |
| `P3` | 여유가 있을 때 진행할 작업 |

---

## 7. Branch 작성 방법

Issue를 만들고 나면 해당 Issue 번호를 포함해서 작업 Branch를 생성합니다.

### 7.1 Branch 이름 형식

```text
타입/이슈번호-작업명
```

예시:

```text
feat/12-face-analyzer
fix/18-camera-reconnect
docs/5-readme-update
chore/3-project-setup
refactor/21-score-engine
test/25-expression-interpreter
ci/30-python-workflow
```

### 7.2 Branch 타입

| 타입 | 설명 |
|---|---|
| `feat` | 새로운 기능 |
| `fix` | 버그 수정 |
| `docs` | 문서 추가 및 수정 |
| `chore` | 설정, 패키지, 프로젝트 구성 |
| `refactor` | 동작 변화 없는 코드 구조 개선 |
| `test` | 테스트 코드 |
| `ci` | GitHub Actions 및 CI 설정 |

---

## 8. 작업 시작 전 Git 명령어

작업 전에는 항상 최신 `main`을 가져옵니다.

```bash
git switch main
git pull origin main
```

작업 Branch를 생성합니다.

```bash
git switch -c feat/12-face-analyzer
```

현재 Branch와 변경 상태를 확인합니다.

```bash
git status
```

---

## 9. Commit 작성 방법

Commit 메시지는 다음 형식을 사용합니다.

```text
type: 작업 내용 (#이슈번호)
```

예시:

```text
feat: MediaPipe 얼굴 지표 추출 구현 (#12)
fix: 카메라 재연결 처리 수정 (#18)
docs: README 실행 방법 추가 (#5)
chore: 프로젝트 초기 구조 생성 (#3)
```

Commit 타입은 Branch 타입과 동일하게 사용합니다.

```bash
git add .
git commit -m "feat: MediaPipe 얼굴 지표 추출 구현 (#12)"
```

---

## 10. Push 방법

작업 Branch를 GitHub에 Push합니다.

```bash
git push -u origin feat/12-face-analyzer
```

Push 후 GitHub Repository에 들어가면 `Compare & pull request` 버튼이 나타납니다.

---

## 11. Pull Request 작성 방법

작업이 끝나면 Pull Request를 생성합니다.

### 11.1 PR 대상

```text
base: main
compare: 내가 작업한 branch
```

예시:

```text
base: main
compare: feat/12-face-analyzer
```

### 11.2 PR 제목

PR 제목은 작업 내용을 명확하게 작성합니다.

예시:

```text
[FEAT] MediaPipe 얼굴 지표 추출 기능 구현
[FIX] 카메라 재연결 오류 수정
[DOCS] Device README 실행 방법 추가
```

### 11.3 PR 본문

```markdown
## 관련 Issue

- Closes #12

## 작업 내용

- MediaPipe Face Landmarker 실행 코드 추가
- blendshape 값을 facial_features 형식으로 변환
- 얼굴 미감지 시 예외 처리

## 테스트 결과

- [x] 사진 1장에서 landmark 출력 확인
- [x] 얼굴이 없는 이미지 입력 시 예외 메시지 출력 확인
- [x] API Key, 비밀번호, 개인정보가 포함되지 않았습니다.

## 리뷰 요청 사항

- facial_features 필드 이름이 Backend API 명세와 맞는지 확인해주세요.
```

`Closes #12`처럼 작성하면 PR이 Merge될 때 해당 Issue가 자동으로 닫힙니다.

---

## 12. Review와 Merge

PR을 올린 뒤 팀원에게 Review를 요청합니다.

리뷰어는 아래 내용을 확인합니다.

- Issue의 완료 조건을 만족하는가
- 불필요하게 큰 변경이 포함되지 않았는가
- 실행 방법이 문서화되어 있는가
- API Key, 비밀번호, 개인정보가 포함되어 있지 않은가
- 기존 기능을 깨뜨리지 않는가
- 에러 상황에 대한 처리가 있는가

리뷰가 끝나면 `Squash and merge`로 병합합니다.

Merge 후 작업 Branch는 삭제합니다.

---

## 13. 작업 중 main이 업데이트된 경우

내 작업 중 다른 사람이 먼저 Merge해서 `main`이 업데이트될 수 있습니다.

그럴 때는 작업 Branch에서 최신 `main`을 반영합니다.

```bash
git switch main
git pull origin main
git switch feat/12-face-analyzer
git merge main
```

충돌이 생기면 충돌 파일을 수정한 뒤:

```bash
git add .
git commit
git push
```

---

## 14. 보안 규칙

아래 파일과 정보는 절대 GitHub에 올리지 않습니다.

- `.env`
- OpenAI API Key
- DB 계정과 비밀번호
- JWT Secret
- Mobius 인증 정보
- 개인키와 인증서
- 사용자의 얼굴 원본 사진 및 영상
- 사용자 개인정보가 포함된 실제 데이터

Repository에는 실제 환경변수 대신 `.env.example`만 등록합니다.

올리기 전에 항상 확인합니다.

```bash
git status
```

---

## 15. Repository별 첫 작업 예시

### 15.1 Device

```text
Issue: [FEAT] MediaPipe 사진 입력 테스트
Branch: feat/1-mediapipe-image-test
Commit: feat: MediaPipe 사진 입력 테스트 구현 (#1)
```

### 15.2 AI

```text
Issue: [FEAT] expression_interpreter 구현
Branch: feat/1-expression-interpreter
Commit: feat: 표정 해석 모듈 구현 (#1)
```

### 15.3 Backend

```text
Issue: [DOCS] Device-BE API 명세 작성
Branch: docs/1-device-be-api
Commit: docs: Device-BE API 명세 작성 (#1)
```

### 15.4 Frontend

```text
Issue: [DESIGN] 결과 화면 와이어프레임 작성
Branch: docs/1-result-wireframe
Commit: docs: 결과 화면 와이어프레임 작성 (#1)
```

---

## 16. 자주 하는 실수

### 16.1 main에서 바로 작업

잘못된 예시:

```bash
git switch main
# 파일 수정
git push origin main
```

올바른 예시:

```bash
git switch main
git pull origin main
git switch -c feat/12-face-analyzer
# 파일 수정
git push -u origin feat/12-face-analyzer
```

### 16.2 Issue 없이 Branch 만들기

작업 전에는 먼저 Issue를 만들고, Issue 번호를 Branch 이름에 포함합니다.

### 16.3 너무 큰 Issue 만들기

좋지 않은 예시:

```text
[FEAT] AI 전체 구현
[FEAT] 백엔드 완성
```

좋은 예시:

```text
[FEAT] expression_interpreter 구현
[FEAT] GPT 질문 생성 프롬프트 작성
[FEAT] AI 서버 응답 스키마 정의
```

### 16.4 .env 업로드

`.env`는 절대 GitHub에 올리지 않습니다.

올려야 하는 것은 `.env.example`입니다.

---

## 17. 전체 요약


```text
1. Issue를 만든다.
2. Issue 번호로 Branch를 만든다.
3. 작업하고 Commit한다.
4. Push한다.
5. Pull Request를 만든다.
6. 팀원 Review를 받는다.
7. Squash and merge한다.
8. Branch를 삭제한다.
```

`main`에는 직접 Push하지 않습니다.
