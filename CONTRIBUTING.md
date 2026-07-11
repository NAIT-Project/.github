# NAIT Contribution Guide

NAIT 프로젝트의 공통 협업 규칙입니다.

## 1. 기본 작업 흐름

모든 작업은 Issue 기반으로 진행합니다.

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

초기 프로젝트 세팅 이후에는 `main` 브랜치에 직접 Push하지 않습니다.

## 2. Repository 구성

| Repository | 역할 |
|---|---|
| `NAIT-Device` | Raspberry Pi, 카메라, MediaPipe, 터치형 아바타 UI |
| `NAIT-AI` | 표정 해석, GPT 추가 질문 생성 |
| `NAIT-BE` | 사용자·기기·세션 관리, DB, 점수 계산, Mobius 연동 |
| `NAIT-FE` | 사용자 결과와 최근 기록을 보여주는 웹페이지 |

## 3. Branch 규칙

Branch 이름은 다음 형식을 따릅니다.

```text
타입/이슈번호-작업명
```

예시:

```text
feat/12-face-analyzer
fix/18-camera-reconnect
docs/5-api-spec
chore/3-project-setup
refactor/21-score-engine
test/25-expression-interpreter
ci/30-python-workflow
```

### Branch 타입

| 타입 | 설명 |
|---|---|
| `feat` | 새로운 기능 |
| `fix` | 버그 수정 |
| `docs` | 문서 추가 및 수정 |
| `chore` | 설정, 패키지, 프로젝트 구성 |
| `refactor` | 동작 변화 없는 코드 구조 개선 |
| `test` | 테스트 코드 |
| `ci` | GitHub Actions 및 CI 설정 |

## 4. Commit 규칙

Commit 메시지는 다음 형식을 사용합니다.

```text
type: 작업 내용 (#이슈번호)
```

예시:

```text
feat: MediaPipe 얼굴 지표 추출 구현 (#12)
fix: 카메라 재연결 처리 수정 (#18)
docs: Device와 BE 사이 API 명세 작성 (#5)
chore: Python 프로젝트 초기 구조 구성 (#3)
```

### Commit 타입

| 타입 | 설명 |
|---|---|
| `feat` | 기능 추가 |
| `fix` | 버그 수정 |
| `docs` | 문서 수정 |
| `refactor` | 코드 구조 개선 |
| `test` | 테스트 추가 및 수정 |
| `style` | 포맷, 들여쓰기 등 |
| `chore` | 설정, 의존성, 기타 작업 |
| `ci` | CI/CD 설정 |

## 5. Issue 작성 규칙

작업 시작 전 Issue를 생성합니다.

Issue 제목은 다음 형식을 사용합니다.

```text
[FEAT] 기능 이름
[FIX] 버그 이름
[DOCS] 문서 작업 이름
[CHORE] 설정 작업 이름
[REFACTOR] 리팩토링 대상
[TEST] 테스트 대상
[CI] CI 작업 이름
```

좋은 예시:

```text
[FEAT] MediaPipe 얼굴 지표 추출 기능 구현
[FIX] 카메라 연결 실패 시 예외 처리
[DOCS] Device 실행 방법 문서 작성
[CHORE] 프로젝트 초기 폴더 구조 생성
```

좋지 않은 예시:

```text
작업
수정
백엔드 완성
AI 만들기
```

Issue 하나는 가능한 한 Pull Request 하나로 끝낼 수 있는 크기로 작성합니다.

## 6. Pull Request 규칙

작업이 끝나면 `main` 브랜치를 대상으로 Pull Request를 생성합니다.

PR 제목은 다음처럼 작성합니다.

```text
[FEAT] MediaPipe 얼굴 지표 추출 기능 구현
[FIX] 카메라 재연결 오류 수정
[DOCS] README 실행 방법 추가
```

PR 본문에는 다음을 포함합니다.

- 관련 Issue
- 작업 내용
- 변경 이유
- 테스트 결과
- 리뷰 요청 사항
- 실행 화면 또는 로그

관련 Issue는 다음처럼 연결합니다.

```text
Closes #12
```

## 7. Review 규칙

리뷰어는 다음 항목을 확인합니다.

- 요구사항을 만족하는가
- 불필요하게 큰 변경이 포함되지 않았는가
- 실행 방법이 문서화되어 있는가
- API Key, 비밀번호, 개인정보가 포함되어 있지 않은가
- 기존 기능을 깨뜨리지 않는가
- 에러 상황에 대한 처리가 있는가

리뷰 의견이 남겨진 경우 작성자는 수정 후 다시 확인 요청을 합니다.

## 8. Merge 규칙

NAIT 프로젝트는 `Squash and merge`를 기본으로 사용합니다.

작업 Branch의 여러 Commit은 `main` 브랜치에 하나의 Commit으로 정리해 병합합니다.

Merge 후 작업 Branch는 삭제합니다.

## 9. 보안 및 개인정보 규칙

다음 항목은 GitHub에 Commit하지 않습니다.

- `.env`
- OpenAI API Key
- DB 계정과 비밀번호
- JWT Secret
- Mobius 인증 정보
- 개인키와 인증서
- 사용자의 얼굴 원본 사진 및 영상
- 사용자 개인정보가 포함된 실제 데이터

Repository에는 실제 환경변수 대신 `.env.example`만 등록합니다.

## 10. 작업 전 확인 명령어

작업 전에는 항상 최신 `main`을 가져옵니다.

```bash
git switch main
git pull origin main
```

새 작업 Branch를 생성합니다.

```bash
git switch -c feat/12-face-analyzer
```

작업 후 변경 사항을 확인합니다.

```bash
git status
```

Commit 후 Push합니다.

```bash
git add .
git commit -m "feat: MediaPipe 얼굴 지표 추출 구현 (#12)"
git push -u origin feat/12-face-analyzer
```
