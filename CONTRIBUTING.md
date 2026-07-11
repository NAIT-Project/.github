
오늘 오후 3:19

AIoT_MoodMirror_System_Architecture(1).pdf
PDF
이런 프로젝트를 진행할 깃, 깃허브 레포를 만들어야 해 팀프로젝트면 organization을 만들고 그 안에 repo를 파면 할 수 있는게 많아져서 좋대 그리고 이슈템플릿이나 pr 템플릿은 초기에 만들어서 넣어두는 게 좋고 보통 서버랑 db를 백엔드 레포로 넣고 프론트 레포 하나 이런 식으로 나눠서 두는 게 좋다고 하는데 지금 팀원들이 맡은 역할 말고도 다른 것도 같이 할 상황인데 그래도 따로 나눠 놓는게 작업 섞이지 않고 좋겠지? '- organization 생성 -> repo 생성 -> 팀원 초대 후 권한 부여
- 컨벤션 정하기 (브랜치 이름, PR, 이슈 형식)
- 컨벤션 토대로 이슈, PR 템플릿 넣어두기
+ 코드 리뷰 에이전트 'coderabbit' 사용하면 좋음
- github actions로 ci/cd 관리하기
- github projects 사용하면 중요도, 일정 관리 파악하기 좋음.' 이렇게 진행할 예정이고 'https://github.com/orgs/GACHI-Project/repositories', 'https://github.com/SWUWEB/DonWorry-BE', 'https://github.com/SWUWEB/DonWorry-FE', 'https://github.com/GACHI-Project/GACHI-AI' 이 organization을 참고해서 만드려고 하는데 어떻게 만들지 차근차근 설명해줘

네. 팀원이 여러 영역을 함께 작업하더라도 레포지토리는 역할 담당자 기준이 아니라, 실행·배포 단위와 기술 스택 기준으로 분리하는 편이 좋습니다.

현재 Mood Mirror 구조에서는 라즈베리파이, 중앙 서버, AI 모듈, 웹페이지가 서로 다른 환경에서 실행됩니다. 특히 자료에서도 라즈베리파이는 데이터 수집과 MediaPipe 분석, 중앙 서버는 사용자·세션·DB·점수 계산, GPT는 질문 생성, 웹은 결과 조회를 담당하도록 나뉘어 있습니다. 


따라서 아래처럼 Organization 1개 + Repository 4개로 시작하는 것을 권장합니다.

1. 추천 GitHub 전체 구조
예를 들어 프로젝트 이름을 MoodMirror라고 하면 다음과 같이 구성합니다.

MoodMirror-Project
├── MoodMirror-Device
├── MoodMirror-BE
├── MoodMirror-AI
├── MoodMirror-FE
└── .github
Repository	담당 범위	예상 기술
MoodMirror-Device	라즈베리파이 UI, 터치 입력, 카메라, MediaPipe	Python, Raspberry Pi
MoodMirror-BE	중앙 API, 사용자·세션 관리, DB, 점수 계산, Mobius 연동	Spring Boot 또는 FastAPI
MoodMirror-AI	표정 해석, GPT 추가 질문 생성	Python, FastAPI, OpenAI API
MoodMirror-FE	사용자 결과 조회 웹페이지	React 또는 Next.js
.github	Organization 공통 템플릿과 안내 문서	Markdown, YAML
GACHI도 GACHI-BE, GACHI-AI, GACHI-FE를 별도 애플리케이션으로 나누고 있으며, AI 서버는 결과 JSON만 반환하고 DB 저장은 백엔드가 담당하는 구조입니다. 이는 현재 Mood Mirror 구조와 상당히 비슷합니다. 

DB는 별도 레포로 만들지 않는 것이 좋습니다
DB 자체를 MoodMirror-DB라는 레포로 분리할 필요는 없습니다.

DB 스키마와 마이그레이션 파일은 백엔드 코드와 함께 변경되는 경우가 많으므로 다음처럼 MoodMirror-BE 안에 넣는 것이 좋습니다.

MoodMirror-BE
├── src/
├── migrations/
│   ├── V1__create_users.sql
│   ├── V2__create_sessions.sql
│   └── V3__create_checkins.sql
├── docs/
│   ├── database.md
│   └── api.md
└── docker-compose.yml
즉:

서버 코드 + DB 접근 코드 + DB 마이그레이션
                  ↓
          MoodMirror-BE
Mobius 연동 코드도 처음에는 백엔드 안에 두는 것이 적절합니다.

MoodMirror-BE
└── src/
    └── integrations/
        └── mobius/
2. 팀원이 여러 영역을 같이 작업해도 분리해야 하는 이유
레포를 나누는 기준은 다음과 같습니다.

누가 개발하는가?        X
어디서 실행되는가?      O
어떻게 배포되는가?      O
기술 스택이 다른가?     O
독립적으로 테스트되는가? O
예를 들어 한 팀원이 라즈베리파이 코드와 AI 코드를 모두 작성하더라도 다음과 같이 작업하면 됩니다.

MoodMirror-Device
└── feat/#12-mediapipe-blendshape

MoodMirror-AI
└── feat/#21-expression-interpreter
이렇게 해야 다음 장점이 있습니다.

라즈베리파이 의존성과 서버 의존성이 섞이지 않습니다.

FE에서 Python 패키지를 설치할 필요가 없습니다.

AI 서버만 따로 실행하고 테스트할 수 있습니다.

한 레포의 오류가 다른 레포의 CI를 깨뜨리지 않습니다.

배포 시 필요한 서비스만 다시 배포할 수 있습니다.

README와 환경변수 설명이 명확해집니다.

다만 초기에 프로젝트가 아주 작다면 AI를 BE 안에 넣어도 됩니다. 현재 구조에서는 expression_interpreter.py, GPT 모듈이 Python 기반이고 독립적으로 발전할 가능성이 높으므로 AI 레포를 따로 두는 쪽이 더 적합합니다.

3. Organization 만들기
GitHub에서 다음 순서로 진행합니다.

프로필 사진
→ Settings
→ Organizations
→ New organization
GitHub Organization은 여러 레포와 프로젝트, 팀, 권한을 한 곳에서 관리하기 위한 기능입니다. 개인 레포에 팀원을 각각 추가하는 것보다 팀 단위 권한 관리가 편합니다. 

Organization 이름
예시는 다음과 같습니다.

MoodMirror-Project
MoodMate-Project
AIoT-MoodMirror
Team-MoodMirror
가장 알아보기 쉬운 것은 다음입니다.

AIoT-MoodMirror
Organization 이름은 GitHub URL에도 사용됩니다.

https://github.com/AIoT-MoodMirror
Organization 생성 시 설정
플랜: 우선 무료 플랜

소유자 이메일: 팀장 또는 프로젝트용 이메일

Organization 소유자: 최소 2명 권장

기본 Repository visibility: 프로젝트 성격에 따라 Private 또는 Public

팀장 한 명만 Owner로 두면 계정 문제가 생겼을 때 설정 변경이 어려울 수 있으므로, 책임 있는 팀원 한 명을 추가 Owner로 두는 것이 안전합니다.

4. 팀원 초대와 Team 만들기
4-1. 팀원 초대
Organization 페이지에서:

People
→ Invite member
→ GitHub 아이디 또는 이메일 입력
→ Member로 초대
GitHub 초대는 일정 기간 내 수락해야 하며, 팀원은 각자 개인 GitHub 계정을 가지고 있어야 합니다. 

4-2. GitHub Team 생성
Organization에서:

Teams
→ New team
다음 Team을 만드는 것을 권장합니다.

maintainers
device
backend
ai
frontend
예:

Team	역할
maintainers	Organization 및 레포 설정 관리
device	라즈베리파이 코드 담당
backend	중앙 서버와 DB 담당
ai	표정 해석과 GPT 연동 담당
frontend	웹페이지 담당
GitHub Team은 팀별 레포 권한 부여와 @AIoT-MoodMirror/backend 같은 멘션에 사용할 수 있습니다. GitHub에서는 독립 Team뿐 아니라 상위·하위 Team도 만들 수 있습니다. 

하지만 팀원이 4~6명 정도라면 처음부터 복잡한 중첩 Team까지 만들 필요는 없습니다.

5. 권한 설정
권한은 다음 정도가 적당합니다.

대상	권한
팀장·GitHub 관리자	Owner
각 파트 팀원	해당 레포 Write
전체 팀원	다른 레포 Read 또는 Write
외부 조언자	Read
CI/CD 담당자	Maintain 또는 Admin
팀원들이 여러 영역을 함께 작업할 예정이라면 모든 개발 팀원에게 네 레포 모두 Write 권한을 줘도 됩니다.

다만 Admin 권한은 제한하는 것이 좋습니다.

Owner/Admin
- 레포 삭제
- Actions Secret 관리
- Branch Rules 변경
- 팀원 권한 변경

Write
- 브랜치 생성
- 코드 Push
- Issue/PR 생성
- PR Merge
GitHub Organization 레포에는 역할별로 세부 권한을 지정할 수 있습니다. 

6. Repository 생성
Organization 메인 화면에서 다음과 같이 생성합니다.

Repositories
→ New repository
다음 네 개를 만듭니다.

MoodMirror-Device
MoodMirror-BE
MoodMirror-AI
MoodMirror-FE
각 레포 생성 시:

Owner: AIoT-MoodMirror

Visibility: Private 또는 Public

Add README: 체크

Add .gitignore: 기술에 맞게 선택

License: 공개 프로젝트일 때 선택

기본 브랜치: 처음에는 main

.gitignore는 다음처럼 설정합니다.

레포	.gitignore
Device	Python
BE	Java 또는 Python
AI	Python
FE	Node
7. 각 Repository 역할 정의
7-1. MoodMirror-Device
MoodMirror-Device/
├── src/
│   ├── face_analyzer.py
│   ├── camera.py
│   ├── api_client.py
│   └── main.py
├── ui/
│   ├── avatar/
│   └── screens/
├── tests/
├── config/
│   └── example.yaml
├── requirements.txt
├── .env.example
└── README.md
담당 기능:

카메라 입력

MediaPipe Face Landmarker

blendshape 추출

고정 자가진단 질문

터치 입력

중앙 서버 전송

결과 화면 표시

원본 얼굴 영상은 외부로 보내지 않고, 라즈베리파이에서 숫자 형태의 표정 지표만 생성하도록 해야 합니다. 


7-2. MoodMirror-AI
MoodMirror-AI/
├── app/
│   ├── main.py
│   ├── api/
│   ├── services/
│   │   ├── expression_interpreter.py
│   │   └── question_generator.py
│   ├── prompts/
│   │   └── adaptive_question.txt
│   └── schemas/
├── tests/
├── requirements.txt
├── Dockerfile
├── .env.example
└── README.md
담당 기능:

facial feature 해석

표정 패턴 라벨 생성

GPT 추가 질문 생성

JSON 결과 반환

DB에는 직접 접근하지 않고 다음처럼 반환하는 것이 좋습니다.

{
  "avatar_message": "수면 상태를 한 번 더 확인해볼게요.",
  "question": "오늘 몸이 무겁거나 무기력한 느낌이 있었나요?",
  "options": [
    "네, 꽤 그랬어요",
    "조금 그랬어요",
    "아니요, 괜찮았어요"
  ]
}
7-3. MoodMirror-BE
MoodMirror-BE/
├── src/
│   ├── users/
│   ├── devices/
│   ├── sessions/
│   ├── checkins/
│   ├── scoring/
│   └── integrations/
│       ├── ai/
│       └── mobius/
├── migrations/
├── docs/
├── tests/
├── docker-compose.yml
├── .env.example
└── README.md
담당 기능:

사용자와 기기 관리

세션 생성

Device 데이터 수신

AI 서버 호출

추가 답변 수신

최종 점수 계산

Central DB 저장

Mobius 저장

FE 조회 API 제공

7-4. MoodMirror-FE
MoodMirror-FE/
├── src/
│   ├── pages/
│   ├── components/
│   ├── api/
│   ├── hooks/
│   └── types/
├── public/
├── tests/
├── package.json
├── .env.example
└── README.md
담당 기능:

로그인

오늘의 컨디션 결과

최근 7일 변화 그래프

세션별 기록

추천 메시지 표시

8. 브랜치 전략 정하기
참고한 GACHI-BE는 기본 브랜치를 develop로 두고, main과 develop에 직접 Push하지 않으며 PR과 CI를 통과한 후 Merge하는 규칙을 사용합니다. 

Mood Mirror에도 다음 전략을 권장합니다.

main
└── 실제 발표·배포 가능한 안정 버전

develop
└── 다음 버전을 통합하는 개발 브랜치

feat/*
fix/*
refactor/*
docs/*
chore/*
test/*
작업 흐름
Issue 생성
→ develop에서 작업 브랜치 생성
→ 코드 작성 및 Commit
→ 원격 브랜치 Push
→ develop 대상으로 PR
→ 코드 리뷰 및 CI
→ Squash and merge
→ 일정 단위로 develop → main PR
브랜치 이름
Issue 번호를 포함하세요.

feat/12-mediapipe-analysis
feat/21-session-api
fix/34-camera-disconnect
refactor/42-score-engine
docs/15-api-specification
chore/8-github-actions
형식:

타입/이슈번호-간단한-설명
한글 브랜치명보다는 영문 소문자를 권장합니다.

9. Commit Convention
다음 형식이 무난합니다.

type: 변경 내용
또는 Issue 번호를 포함해서:

type: 변경 내용 (#이슈번호)
예:

feat: MediaPipe blendshape 추출 기능 구현 (#12)
fix: 카메라 연결 해제 시 재연결 처리 (#18)
docs: 중앙 서버 API 명세 추가 (#24)
refactor: 감정 점수 계산 로직 분리 (#31)
test: 표정 해석 모듈 테스트 추가 (#35)
chore: Python CI workflow 추가 (#40)
타입은 다음 정도만 사용하면 됩니다.

Type	의미
feat	새로운 기능
fix	버그 수정
refactor	기능 변화 없는 구조 개선
docs	문서 변경
test	테스트
style	포맷·코드 스타일
chore	설정·빌드·패키지
ci	GitHub Actions 변경
10. Issue Convention
Issue 제목은 다음처럼 작성합니다.

[FEAT] MediaPipe 얼굴 지표 추출
[FIX] 카메라 재연결 오류 수정
[DOCS] Device-Server 데이터 명세 작성
[REFACTOR] 점수 계산 모듈 분리
Issue 하나는 가능한 한 PR 하나로 완료할 수 있는 크기로 만듭니다.

좋지 않은 예:

AI 전체 개발
백엔드 완성
웹페이지 제작
좋은 예:

MediaPipe 사진 입력 테스트
blendshape 결과 dict 변환
표정 지표 API Schema 정의
GPT 추가 질문 프롬프트 작성
세션 생성 API 구현
최근 7일 결과 API 구현
11. Issue Template 만들기
각 레포에 다음 경로를 생성합니다.

.github/
└── ISSUE_TEMPLATE/
    ├── feature.yml
    ├── bug.yml
    └── config.yml
.github/ISSUE_TEMPLATE/feature.yml
name: 기능 개발
description: 새로운 기능 개발 작업을 등록합니다.
title: "[FEAT] "
labels:
  - feature
body:
  - type: textarea
    id: description
    attributes:
      label: 기능 설명
      description: 구현할 기능을 구체적으로 작성해주세요.
      placeholder: MediaPipe의 blendshape 값을 표정 지표 dict로 변환합니다.
    validations:
      required: true

  - type: textarea
    id: tasks
    attributes:
      label: 작업 내용
      value: |
        - [ ] 세부 작업 1
        - [ ] 세부 작업 2
        - [ ] 테스트 작성
        - [ ] 문서 업데이트
    validations:
      required: true

  - type: textarea
    id: acceptance
    attributes:
      label: 완료 조건
      placeholder: 함수 실행 시 정의된 facial_features JSON이 반환됩니다.
    validations:
      required: true

  - type: textarea
    id: reference
    attributes:
      label: 참고 자료
      description: 관련 문서, 화면, API 또는 Issue를 작성해주세요.
.github/ISSUE_TEMPLATE/bug.yml
name: 버그 보고
description: 발생한 오류를 등록합니다.
title: "[FIX] "
labels:
  - bug
body:
  - type: textarea
    id: problem
    attributes:
      label: 문제 설명
      description: 어떤 문제가 발생했는지 작성해주세요.
    validations:
      required: true

  - type: textarea
    id: reproduction
    attributes:
      label: 재현 방법
      value: |
        1.
        2.
        3.
    validations:
      required: true

  - type: textarea
    id: expected
    attributes:
      label: 기대 동작
    validations:
      required: true

  - type: textarea
    id: environment
    attributes:
      label: 실행 환경
      placeholder: |
        OS:
        Python/Node/Java 버전:
        Raspberry Pi 모델:
        Browser:
    validations:
      required: true

  - type: textarea
    id: logs
    attributes:
      label: 로그 및 화면
      description: 오류 로그나 스크린샷을 첨부해주세요.
      render: shell
.github/ISSUE_TEMPLATE/config.yml
blank_issues_enabled: false
contact_links: []
12. PR Template 만들기
레포마다 다음 파일을 만듭니다.

.github/
└── pull_request_template.md
내용:

## 관련 Issue

- Closes #

## 변경 내용

- 
- 
- 

## 변경 이유

이 변경이 필요한 이유를 작성해주세요.

## 테스트

- [ ] 로컬에서 정상 실행되는지 확인했습니다.
- [ ] 기존 기능이 정상 작동하는지 확인했습니다.
- [ ] 필요한 테스트 코드를 추가했습니다.
- [ ] 민감한 정보가 포함되지 않았는지 확인했습니다.

## 확인이 필요한 부분

리뷰어가 집중해서 확인해야 할 내용을 작성해주세요.

## 스크린샷 또는 실행 결과

UI 변경이나 실행 결과가 있다면 첨부해주세요.
Closes #12처럼 작성하면 PR이 Merge될 때 관련 Issue도 함께 닫을 수 있습니다.

13. Organization 공통 .github 레포
모든 레포에 동일한 템플릿을 복사해도 되지만, Organization에 다음 이름의 레포를 하나 만들면 공통 Community 파일을 관리하기 좋습니다.

.github
구조:

.github/
├── profile/
│   └── README.md
├── ISSUE_TEMPLATE/
│   ├── feature.yml
│   └── bug.yml
├── pull_request_template.md
├── CONTRIBUTING.md
└── CODE_OF_CONDUCT.md
profile/README.md에는 Organization 첫 화면 소개를 작성합니다.

# AIoT Mood Mirror

AI 아바타 기반 감정 컨디션 모니터링 프로젝트입니다.

## Repositories

- MoodMirror-Device: Raspberry Pi 및 MediaPipe
- MoodMirror-BE: 중앙 서버, DB, Mobius
- MoodMirror-AI: 표정 해석 및 GPT 질문 생성
- MoodMirror-FE: 사용자 결과 조회 웹페이지

## Architecture

Raspberry Pi → Central Server → AI Server
                         → Central DB
                         → Mobius
                         → Web
다만 레포마다 완료 조건이나 체크리스트가 다르다면 해당 레포 안의 템플릿이 우선되도록 개별 템플릿을 두는 것이 좋습니다.

14. Labels 만들기
모든 레포에서 Label을 통일합니다.

작업 종류
feature
bug
refactor
docs
test
chore
ci
design
영역
device
backend
ai
frontend
database
mobius
openai
우선순위
priority: high
priority: medium
priority: low
진행 상태
GitHub Projects에서 상태를 관리할 예정이라면 Issue Label에는 상태 Label을 너무 많이 만들지 않는 것이 좋습니다.

15. GitHub Projects 설정
Organization 화면에서:

Projects
→ New project
→ Board 또는 Table 선택
프로젝트 이름:

MoodMirror Development
추천 필드는 다음과 같습니다.

Field	값
Status	Backlog, Ready, In Progress, Review, Done
Priority	P0, P1, P2, P3
Repository	Device, BE, AI, FE
Area	MediaPipe, Server, GPT, Mobius, Web
Iteration	1주차, 2주차 또는 Sprint 1, Sprint 2
Assignee	담당자
Start date	시작일
Target date	마감일
추천 Board 구조
Backlog
→ Ready
→ In Progress
→ Review
→ Done
의미:

Backlog: 나중에 할 작업

Ready: 요구사항이 확정되어 바로 시작 가능

In Progress: 작업 중

Review: PR 리뷰 또는 테스트 중

Done: Merge 및 확인 완료

한 사람이 여러 파트를 맡더라도 프로젝트에서는 모든 레포의 Issue를 한 화면에서 볼 수 있습니다.

16. Milestone 설정
발표나 공모전 일정 기준으로 Milestone을 만듭니다.

예:

M1 - MediaPipe Prototype
M2 - Device and Server Integration
M3 - GPT Adaptive Question
M4 - Mobius Integration
M5 - Web Dashboard
M6 - Final Demo
현재 구현 순서에 맞춰 다음과 같이 연결할 수 있습니다.

M1
- MediaPipe 단독 테스트
- face_analyzer.py 구현

M2
- expression_interpreter.py
- 터치형 아바타 UI
- 중앙 서버 API

M3
- GPT 추가 질문 생성
- score_engine.py

M4
- Mobius AE/CNT/CIN 저장

M5
- 사용자별 결과 조회 웹페이지
이는 업로드한 설계 문서의 구현 순서와도 일치합니다. 


17. Branch Protection 또는 Ruleset 설정
각 레포의 다음 메뉴에서 설정합니다.

Repository
→ Settings
→ Rules
→ Rulesets
→ New branch ruleset
또는 기존 Branch protection 메뉴를 사용할 수 있습니다.

대상 브랜치:

main
develop
main 권장 규칙
Require a pull request before merging
Required approvals: 1
Dismiss stale approvals
Require conversation resolution
Require status checks to pass
Block force pushes
Block deletions
develop 권장 규칙
Require a pull request before merging
Required approvals: 1
Require conversation resolution
Require status checks to pass
Block force pushes
GitHub의 보호 규칙을 사용하면 PR 승인, 상태 검사 통과, 리뷰 대화 해결 등을 Merge 조건으로 지정할 수 있습니다. 

초기에는 Require status checks를 바로 켜지 말고, GitHub Actions를 한 번 실행해 검사 이름이 생성된 뒤 설정하는 것이 편합니다.

18. Merge 방식 설정
Repository
→ Settings
→ General
→ Pull Requests
다음 설정을 권장합니다.

Allow squash merging       체크
Allow merge commits        해제
Allow rebase merging       선택
Automatically delete head branches 체크
작은 기능 브랜치의 여러 Commit을 하나로 정리하기 위해 Squash merge를 기본으로 사용하면 이력이 깔끔합니다.

PR 제목이 최종 Commit 메시지가 되므로 다음처럼 작성합니다.

[FEAT] MediaPipe 표정 지표 추출 기능 구현
19. CODEOWNERS 설정
각 레포에 다음 파일을 만들 수 있습니다.

.github/CODEOWNERS
예를 들어 AI 레포:

* @AIoT-MoodMirror/ai

/app/prompts/ @AIoT-MoodMirror/ai
/.github/ @AIoT-MoodMirror/maintainers
/Dockerfile @AIoT-MoodMirror/maintainers
백엔드:

* @AIoT-MoodMirror/backend

/src/integrations/mobius/ @AIoT-MoodMirror/backend
/migrations/ @AIoT-MoodMirror/backend
/.github/ @AIoT-MoodMirror/maintainers
CODEOWNERS와 Branch Rule을 함께 설정하면 지정된 팀의 리뷰를 Merge 조건으로 요구할 수 있습니다. 

팀원이 적다면 처음부터 필수 리뷰어로 강제하기보다는 자동 리뷰 요청 용도로만 사용해도 됩니다.

20. GitHub Actions CI 설정
CI는 처음부터 간단하게 넣고, 배포 CD는 서비스가 완성된 뒤 추가하는 것이 좋습니다.

20-1. Device·AI Python CI
파일:

.github/workflows/ci.yml
name: Python CI

on:
  pull_request:
    branches:
      - develop
      - main
  push:
    branches:
      - develop
      - main

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"
          cache: pip

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Lint
        run: |
          pip install ruff
          ruff check .

      - name: Test
        run: |
          pip install pytest
          pytest
라즈베리파이 전용 카메라나 GPIO 테스트는 GitHub 서버에서 직접 실행하기 어렵기 때문에 다음처럼 분리합니다.

단위 테스트
- GitHub Actions에서 실행

카메라·GPIO·터치 테스트
- 실제 Raspberry Pi에서 실행
20-2. FE CI
name: Frontend CI

on:
  pull_request:
    branches:
      - develop
      - main

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: npm

      - run: npm ci
      - run: npm run lint
      - run: npm run build
      - run: npm test -- --run
20-3. BE CI
Spring Boot라면:

name: Backend CI

on:
  pull_request:
    branches:
      - develop
      - main

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: "21"
          cache: gradle

      - run: chmod +x gradlew
      - run: ./gradlew test
      - run: ./gradlew build
GitHub Actions는 레포별로 허용되는 Action과 Workflow 정책을 관리할 수 있습니다. 

21. Secret 관리
다음 파일은 절대로 Commit하면 안 됩니다.

.env
service-account.json
private-key.pem
OpenAI API Key
DB password
Mobius 인증 정보
JWT secret
레포에는 .env.example만 올립니다.

OPENAI_API_KEY=
DATABASE_URL=
MOBIUS_BASE_URL=
MOBIUS_ORIGIN=
JWT_SECRET=
실제 값은 다음 메뉴에 등록합니다.

Repository
→ Settings
→ Secrets and variables
→ Actions
→ New repository secret
예:

OPENAI_API_KEY
DATABASE_URL
MOBIUS_ORIGIN
DOCKERHUB_USERNAME
DOCKERHUB_TOKEN
22. CodeRabbit 설치
CodeRabbit은 Organization 전체에 무조건 설치하기보다 처음에는 네 프로젝트 레포만 선택해서 설치하는 것이 좋습니다.

CodeRabbit GitHub App 설치
→ Organization 선택
→ Only select repositories
→ Device, BE, AI, FE 선택
→ Install & Authorize
설치 후 PR을 만들면 자동 리뷰를 실행할 수 있습니다. 설정은 각 레포 루트의 .coderabbit.yaml로 조정할 수 있으며, 기본 설정에서는 대상 브랜치에 열린 적합한 PR을 자동 리뷰하고 Draft PR은 기본적으로 제외합니다. 

.coderabbit.yaml 예시
language: ko-KR

reviews:
  profile: assertive

  auto_review:
    enabled: true
    drafts: false
    base_branches:
      - develop
      - main

  request_changes_workflow: false

  path_instructions:
    - path: "app/prompts/**"
      instructions: >
        프롬프트가 의료 진단을 단정적으로 표현하지 않는지 검토하세요.
        사용자의 감정이나 정신건강 상태를 확정적으로 판단하는 표현을 지적하세요.

    - path: "src/integrations/mobius/**"
      instructions: >
        Mobius AE/CNT/CIN 요청 구조와 오류 처리,
        인증 정보 노출 가능성을 집중적으로 검토하세요.

    - path: "**/*.py"
      instructions: >
        타입 힌트, 예외 처리, 입력값 검증 및 테스트 누락을 검토하세요.
CodeRabbit은 GitHub Actions 등의 Check 실패 결과도 참고하여 PR에 수정 제안을 제공할 수 있습니다. 

단, CodeRabbit 승인을 사람의 리뷰 대체물로 사용하지는 마세요. 최종 Merge는 팀원 한 명이 코드와 실행 결과를 확인한 후 진행하는 것이 좋습니다.

23. 여러 레포 간 API 명세 관리
레포를 분리할 때 가장 자주 생기는 문제는 다음과 같습니다.

Device는 mood_score라고 전송
BE는 moodScore를 기대
AI는 mood라고 기대
이를 막기 위해 API 명세를 먼저 정해야 합니다.

초기에는 MoodMirror-BE/docs에 관리해도 됩니다.

MoodMirror-BE/docs/
├── device-api.md
├── ai-api.md
├── frontend-api.md
├── mobius-resource.md
└── error-code.md
예:

POST /api/v1/sessions/{sessionId}/analysis

Request:
{
  "device_uid": "moodmirror_pi_001",
  "self_report": {
    "mood_score": 2,
    "sleep_hours": 5,
    "stress_score": 4,
    "activity_level": "low"
  },
  "facial_features": {
    "smile_intensity": 0.12,
    "blink_intensity": 0.06
  }
}
GACHI-BE도 공통 응답 형식, 오류 코드, 환경변수, 배포, DB 마이그레이션 규칙을 docs 디렉터리에 구분하여 관리하고 있습니다. 

가능하면 나중에 OpenAPI/Swagger 명세를 사용하세요.

24. 처음 등록할 Issue 목록
Organization과 레포를 만든 직후 다음 Issue들을 등록하면 됩니다.

Device
[CHORE] Raspberry Pi Python 프로젝트 초기 설정
[FEAT] MediaPipe 사진 입력 테스트
[FEAT] face_analyzer.py 구현
[FEAT] 표정 지표 JSON 변환
[FEAT] 고정 자가진단 UI 구현
[FEAT] 중앙 서버 API 연결
AI
[CHORE] FastAPI 프로젝트 초기 설정
[FEAT] 표정 해석 규칙 정의
[FEAT] expression_interpreter.py 구현
[FEAT] GPT 질문 생성 프롬프트 작성
[FEAT] 구조화 JSON 응답 구현
[TEST] 의료 진단 표현 방지 테스트
BE
[CHORE] 중앙 서버 프로젝트 초기 설정
[DOCS] Device-BE API 명세 작성
[DOCS] BE-AI API 명세 작성
[FEAT] 사용자 및 기기 테이블 생성
[FEAT] 세션 생성 API 구현
[FEAT] 분석 데이터 수신 API 구현
[FEAT] AI 서버 호출 구현
[FEAT] 최종 점수 계산 구현
[FEAT] Mobius 저장 모듈 구현
FE
[CHORE] React 프로젝트 초기 설정
[DESIGN] 결과 화면 와이어프레임
[FEAT] 오늘의 컨디션 화면 구현
[FEAT] 최근 7일 그래프 구현
[FEAT] 세션 기록 조회 구현
25. 실제 초기 구축 순서
아래 순서대로 하면 가장 덜 꼬입니다.

1단계: 관리자 구조
1. Organization 생성
2. Owner 2명 지정
3. 전체 팀원 초대
4. Teams 생성
2단계: Repository
5. Device 레포 생성
6. BE 레포 생성
7. AI 레포 생성
8. FE 레포 생성
9. .github 레포 생성
3단계: 협업 규칙
10. README 작성
11. CONTRIBUTING.md 작성
12. 브랜치·Commit·PR Convention 확정
13. Issue Template 추가
14. PR Template 추가
15. Label 생성
4단계: 보호 규칙
16. develop 브랜치 생성
17. main/develop Ruleset 설정
18. PR 승인 1명 요구
19. 직접 Push 금지
20. 자동 브랜치 삭제 설정
5단계: 작업 관리
21. Organization Project 생성
22. Status, Priority, Area, Iteration 필드 생성
23. Milestone 생성
24. 초기 Issue 등록
25. 담당자와 일정 지정
6단계: 자동화
26. 각 레포 기본 GitHub Actions 추가
27. CI가 한 번 정상 실행되는지 확인
28. CI를 Required status check로 지정
29. CodeRabbit 설치
30. 첫 PR로 전체 동작 확인
26. 프로젝트에 적용할 최종 권장안
Organization
└── AIoT-MoodMirror

Repositories
├── MoodMirror-Device
├── MoodMirror-BE
├── MoodMirror-AI
├── MoodMirror-FE
└── .github

Branches
├── main
├── develop
└── feat/#-description

Merge
└── Squash merge

Required
├── PR 1명 승인
├── CI 통과
├── 리뷰 대화 해결
└── main/develop 직접 Push 금지

Project Status
├── Backlog
├── Ready
├── In Progress
├── Review
└── Done
핵심은 레포를 팀원별로 나누지 않고 시스템 경계별로 나누는 것입니다. 한 팀원이 Device와 AI를 동시에 맡아도 각각의 레포에서 Issue와 브랜치를 만들어 작업하면 됩니다. 현재 프로젝트에서는 Device / BE / AI / FE 4개 분리가 과하지 않으면서도 구조가 가장 명확합니다.


초기 세팅으로 파일 만들 때 지금 팀원이 없긴 한데 main에 바로 커밋해도 돼?

25s 동안 처리함
GACHI는 BE / AI / FE / .github를 분리하고, 각 레포에 README.md, .env.example, .gitignore, .editorconfig, .gitattributes, docs/, .coderabbit.yaml 등을 두는 구조입니다. AI는 FastAPI로 분석 결과만 반환하고 DB 저장은 BE에서 담당하며, BE는 Spring Boot와 DB·배포 문서를 관리합니다. 

NAIT도 이를 참고하되, 현재는 GitHub를 처음 사용하는 팀이므로 Docker·CI/CD·CodeRabbit까지 한 번에 넣지 말고 아래 초기 파일부터 완성하는 것이 좋습니다. 프로젝트 설계상 Device는 라즈베리파이와 MediaPipe, AI는 표정 해석·GPT, BE는 사용자·세션·DB·Mobius, FE는 결과 조회를 맡도록 구분합니다. 


1. 최종 초기 구조
현재 만든 네 레포에 다음 파일을 넣으세요.

NAIT-Device
├── README.md
├── .gitignore
├── .env.example
├── .editorconfig
├── .gitattributes
└── docs/
    └── architecture.md

NAIT-AI
├── README.md
├── .gitignore
├── .env.example
├── .editorconfig
├── .gitattributes
└── docs/
    ├── env.md
    └── api.md

NAIT-BE
├── README.md
├── .gitignore
├── .env.example
├── .editorconfig
├── .gitattributes
└── docs/
    ├── api-response.md
    ├── error-code.md
    ├── env.md
    └── database.md

NAIT-FE
├── README.md
├── .gitignore
├── .env.example
├── .editorconfig
├── .gitattributes
└── docs/
    └── ui.md
그리고 Organization에 공통 설정용 레포를 추가합니다.

.github
├── profile/
│   └── README.md
├── ISSUE_TEMPLATE/
│   ├── feature.yml
│   ├── bug.yml
│   ├── task.yml
│   └── config.yml
├── pull_request_template.md
└── CONTRIBUTING.md
.github 레포는 이름을 정확하게 .github로 만들어야 합니다.

2. Organization .github 레포
2-1. profile/README.md
Organization 첫 화면에 표시되는 소개입니다.

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
Repositories
Repository	역할
NAIT-Device	Raspberry Pi, 카메라, MediaPipe, 터치형 아바타 UI
NAIT-AI	표정 해석, GPT 추가 질문 생성
NAIT-BE	사용자·기기·세션 관리, DB, 점수 계산, Mobius 연동
NAIT-FE	사용자 결과와 최근 기록을 보여주는 웹페이지
Collaboration
모든 기능 개발은 다음 순서로 진행합니다.

Issue 생성
→ 작업 Branch 생성
→ Commit 및 Push
→ Pull Request 생성
→ Review
→ Squash Merge
main 브랜치에는 직접 Push하지 않습니다.


---

## 2-2. `CONTRIBUTING.md`

팀 전체 Git 사용 규칙입니다.

```markdown
# NAIT Contribution Guide

NAIT 프로젝트의 공통 협업 규칙입니다.

## 1. 작업 절차

모든 작업은 다음 순서로 진행합니다.

1. 작업할 Repository에서 Issue를 생성합니다.
2. Issue 번호를 포함한 작업 Branch를 생성합니다.
3. 작업 단위별로 Commit합니다.
4. 작업 완료 후 `main` 브랜치를 대상으로 Pull Request를 생성합니다.
5. 팀원 Review와 확인 후 Squash Merge합니다.
6. Merge가 완료된 작업 Branch는 삭제합니다.

## 2. Branch 규칙

형식:

```text
타입/이슈번호-작업명
예시:

feat/12-face-analyzer
fix/18-camera-reconnect
docs/5-api-spec
chore/3-project-setup
refactor/21-score-engine
test/25-expression-interpreter
ci/30-python-workflow
Branch 타입
타입	설명
feat	새로운 기능
fix	버그 수정
docs	문서 추가 및 수정
chore	설정, 패키지, 프로젝트 구성
refactor	동작 변화 없는 코드 구조 개선
test	테스트 코드
ci	GitHub Actions 및 CI 설정
3. Commit 규칙
형식:

type: 작업 내용 (#이슈번호)
예시:

feat: MediaPipe 얼굴 지표 추출 구현 (#12)
fix: 카메라 재연결 처리 수정 (#18)
docs: Device와 BE 사이 API 명세 작성 (#5)
chore: Python 프로젝트 초기 구조 구성 (#3)
Commit 타입
feat: 기능 추가

fix: 버그 수정

docs: 문서 수정

refactor: 코드 구조 개선

test: 테스트 추가 및 수정

style: 포맷, 들여쓰기 등

chore: 설정, 의존성, 기타 작업

ci: CI/CD 설정

4. Pull Request 규칙
PR 하나에는 하나의 주요 목적만 포함합니다.

PR 제목은 변경 내용을 명확하게 작성합니다.

관련 Issue를 Closes #번호로 연결합니다.

실행 결과나 UI 변경이 있으면 화면 또는 로그를 첨부합니다.

API Key, 비밀번호, 개인정보가 포함되지 않았는지 확인합니다.

Review 의견을 모두 해결한 후 Merge합니다.

5. 보안 및 개인정보
다음 항목은 GitHub에 Commit하지 않습니다.

.env

OpenAI API Key

DB 계정과 비밀번호

JWT Secret

Mobius 인증 정보

개인키와 인증서

사용자의 얼굴 원본 사진 및 영상

사용자 개인정보가 포함된 실제 데이터

Repository에는 실제 환경변수 대신 .env.example만 등록합니다.

6. Merge 방식
NAIT 프로젝트는 Squash and merge를 기본으로 사용합니다.

작업 Branch의 여러 Commit은 main에 하나의 정리된 Commit으로 병합합니다.


GACHI도 `main`, `develop` 직접 Push를 피하고 PR과 CI를 통해 병합하며, `feat`, `fix`, `refactor`, `docs`, `style`, `chore` 등의 타입을 사용합니다. NAIT는 초기에 혼동을 줄이기 위해 `develop` 없이 `main + 작업 브랜치`부터 시작하는 구성이 적절합니다. :contentReference[oaicite:2]{index=2}

---

# 3. Issue Template

## 3-1. `ISSUE_TEMPLATE/feature.yml`

```yaml
name: 기능 개발
description: 새로운 기능 구현 작업을 등록합니다.
title: "[FEAT] "
labels:
  - feature
body:
  - type: dropdown
    id: area
    attributes:
      label: 작업 영역
      description: 이 기능이 속한 영역을 선택해주세요.
      options:
        - Device
        - AI
        - Backend
        - Frontend
        - Database
        - Mobius
        - Common
    validations:
      required: true

  - type: textarea
    id: description
    attributes:
      label: 기능 설명
      description: 구현할 기능과 필요한 이유를 작성해주세요.
      placeholder: |
        MediaPipe blendshape 값을 받아서
        smile_intensity, blink_intensity 등의 표정 지표로 변환합니다.
    validations:
      required: true

  - type: textarea
    id: tasks
    attributes:
      label: 작업 항목
      value: |
        - [ ] 요구사항 확인
        - [ ] 기능 구현
        - [ ] 입력값 및 예외 처리
        - [ ] 테스트
        - [ ] 문서 수정
    validations:
      required: true

  - type: textarea
    id: acceptance
    attributes:
      label: 완료 조건
      description: 어떤 상태가 되면 작업을 완료한 것으로 볼지 작성해주세요.
      placeholder: |
        face_analyzer 실행 시 facial_features 형식의 dict가 반환됩니다.
        얼굴이 없는 경우에도 프로그램이 종료되지 않습니다.
    validations:
      required: true

  - type: textarea
    id: reference
    attributes:
      label: 참고 자료
      description: 관련 문서, API, 디자인, 선행 Issue를 작성해주세요.
GACHI의 기능 Issue도 작업 요약, 작업 항목, 완료 조건을 구분하여 관리합니다. 

3-2. ISSUE_TEMPLATE/bug.yml
name: 버그 보고
description: 오류 및 예상과 다른 동작을 등록합니다.
title: "[FIX] "
labels:
  - bug
body:
  - type: dropdown
    id: area
    attributes:
      label: 발생 영역
      options:
        - Device
        - AI
        - Backend
        - Frontend
        - Database
        - Mobius
        - Common
    validations:
      required: true

  - type: textarea
    id: problem
    attributes:
      label: 문제 설명
      description: 발생한 문제를 구체적으로 작성해주세요.
    validations:
      required: true

  - type: textarea
    id: reproduction
    attributes:
      label: 재현 방법
      value: |
        1.
        2.
        3.
    validations:
      required: true

  - type: textarea
    id: expected
    attributes:
      label: 기대한 동작
      description: 정상이라면 어떻게 동작해야 하는지 작성해주세요.
    validations:
      required: true

  - type: textarea
    id: environment
    attributes:
      label: 실행 환경
      value: |
        - OS:
        - Device:
        - Python / Java / Node 버전:
        - Browser:
        - 실행 Branch:
    validations:
      required: true

  - type: textarea
    id: logs
    attributes:
      label: 로그 또는 화면
      description: API Key와 개인정보는 제거하고 첨부해주세요.
3-3. ISSUE_TEMPLATE/task.yml
기능 구현 외 설정·조사·문서 작업용입니다.

name: 일반 작업
description: 프로젝트 설정, 조사, 문서, 운영 작업을 등록합니다.
title: "[TASK] "
labels:
  - chore
body:
  - type: dropdown
    id: branch_type
    attributes:
      label: Branch 타입
      options:
        - chore
        - docs
        - refactor
        - test
        - ci
    validations:
      required: true

  - type: textarea
    id: summary
    attributes:
      label: 작업 요약
      description: 작업 목적과 필요한 이유를 작성해주세요.
    validations:
      required: true

  - type: textarea
    id: tasks
    attributes:
      label: 작업 항목
      value: |
        - [ ] 작업 내용 1
        - [ ] 작업 내용 2
        - [ ] 확인 및 문서화
    validations:
      required: true

  - type: textarea
    id: acceptance
    attributes:
      label: 완료 조건
      description: 작업 완료를 판단할 수 있는 기준을 작성해주세요.
    validations:
      required: true
GACHI에서도 서버 구조, CodeRabbit, CI/CD, 문서 정리와 같은 초기 설정 작업을 별도 Task Issue로 관리했습니다. 

3-4. ISSUE_TEMPLATE/config.yml
blank_issues_enabled: true
contact_links: []
처음에는 빈 Issue도 허용하는 true가 좋습니다. 팀이 익숙해진 뒤 false로 바꿀 수 있습니다.

4. pull_request_template.md
## 관련 Issue

- Closes #

## 작업 내용

- 
- 
- 

## 변경 이유

이 변경이 필요한 이유를 작성해주세요.

## 주요 변경 파일

| 파일 또는 경로 | 변경 내용 |
|---|---|
|  |  |

## 테스트 결과

- [ ] 직접 실행해 정상 동작을 확인했습니다.
- [ ] 기존 기능에 영향을 주지 않는지 확인했습니다.
- [ ] 필요한 예외 처리를 확인했습니다.
- [ ] 필요한 테스트 코드를 추가하거나 수정했습니다.
- [ ] API Key, 비밀번호, 개인정보가 포함되지 않았습니다.
- [ ] 관련 문서를 수정했습니다.

## 리뷰 요청 사항

리뷰어가 집중해서 확인해야 할 부분을 작성해주세요.

## 실행 화면 또는 로그

UI 변경, API 응답, 테스트 결과가 있다면 첨부해주세요.
이 파일을 현재 화면처럼 다음 경로에 만들면 됩니다.

.github/pull_request_template.md
Organization 공통 .github 레포에 넣으면 개별 레포에 템플릿이 없는 경우 공통 템플릿을 사용할 수 있습니다. 지금 이미 NAIT-Device에 만들고 있다면 그대로 두어도 됩니다.

5. 공통 .editorconfig
네 레포 모두 동일한 내용을 사용합니다.

root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
indent_style = space
indent_size = 4

[*.{yml,yaml,json,md,js,jsx,ts,tsx}]
indent_size = 2

[*.md]
trim_trailing_whitespace = false

[Makefile]
indent_style = tab
역할은 팀원마다 IDE가 달라도 들여쓰기와 줄바꿈 형식을 통일하는 것입니다.

6. 공통 .gitattributes
* text=auto eol=lf

*.sh text eol=lf
*.py text eol=lf
*.java text eol=lf
*.kt text eol=lf
*.js text eol=lf
*.jsx text eol=lf
*.ts text eol=lf
*.tsx text eol=lf
*.yml text eol=lf
*.yaml text eol=lf
*.md text eol=lf

*.bat text eol=crlf
*.cmd text eol=crlf
*.ps1 text eol=crlf

*.png binary
*.jpg binary
*.jpeg binary
*.gif binary
*.webp binary
*.pdf binary
*.zip binary
Windows와 Linux를 함께 사용해도 줄바꿈 차이로 파일 전체가 변경된 것처럼 나타나는 문제를 줄여줍니다.

7. NAIT-Device
7-1. README.md
# NAIT-Device

NAIT 프로젝트의 Raspberry Pi 디바이스 애플리케이션입니다.

라즈베리파이에서 아바타 UI, 터치 입력, 카메라 입력 및
MediaPipe 얼굴 분석을 수행합니다.

원본 얼굴 영상은 외부 서버로 전송하지 않으며,
Device 내부에서 분석한 숫자 형태의 얼굴 지표만 중앙 서버로 전송합니다.

## 주요 역할

- Raspberry Pi 카메라 입력
- MediaPipe Face Landmarker 실행
- 얼굴 landmark 및 blendshape 추출
- 표정 지표 생성
- 고정 자가진단 질문 표시
- 터치 답변 수집
- 중앙 서버 API 호출
- 추가 질문 및 최종 결과 표시

## 처리 흐름

```text
Camera Input
  ↓
MediaPipe Face Landmarker
  ↓
Facial Feature Extraction
  ↓
Self-report + Facial Features
  ↓ HTTPS
Central Server
전송 데이터 예시
{
  "device_uid": "nait_pi_001",
  "session_id": "20260711_001",
  "self_report": {
    "mood_score": 2,
    "sleep_hours": 5,
    "stress_score": 4,
    "activity_level": "low"
  },
  "facial_features": {
    "smile_intensity": 0.12,
    "frown_intensity": 0.03,
    "blink_intensity": 0.06,
    "brow_tension": 0.09,
    "eye_engagement": 0.04,
    "expression_variability": 0.02
  }
}
예정 기술
Python

Raspberry Pi

MediaPipe

OpenCV

Touch Display

예상 구조
NAIT-Device/
├── app/
│   ├── main.py
│   ├── camera/
│   ├── analysis/
│   │   └── face_analyzer.py
│   ├── ui/
│   └── api/
│       └── server_client.py
├── tests/
├── docs/
├── requirements.txt
├── .env.example
└── README.md
로컬 실행
프로젝트 코드가 추가된 뒤 아래 방식으로 실행합니다.

python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python -m app.main
Windows PowerShell:

python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
python -m app.main
환경변수
.env.example을 복사해 .env 파일을 생성합니다.

cp .env.example .env
실제 API Key나 비밀번호가 포함된 .env는 Commit하지 않습니다.

작업 규칙
기본 브랜치: main

작업 브랜치: feat/이슈번호-작업명

main 직접 Push 금지

Pull Request Review 후 Squash Merge

예시:

feat/12-face-analyzer
fix/18-camera-reconnect
chore/3-device-setup

## 7-2. `.env.example`

```env
APP_ENV=development
LOG_LEVEL=INFO

DEVICE_UID=nait_pi_001
CAMERA_INDEX=0

SERVER_BASE_URL=http://localhost:8080
SERVER_REQUEST_TIMEOUT=10

MEDIAPIPE_MODEL_PATH=./models/face_landmarker.task
7-3. .gitignore
# Python
__pycache__/
*.py[cod]
*.pyd
*.pyo
.pytest_cache/
.mypy_cache/
.ruff_cache/
.coverage
htmlcov/

# Virtual environments
.venv/
venv/
env/

# Environment variables
.env
.env.*
!.env.example

# IDE
.vscode/
.idea/
*.iml

# OS
.DS_Store
Thumbs.db

# Logs and generated output
logs/
outputs/
reports/
*.log

# Camera and personal data
captures/
recordings/
screenshots/
user_data/
face_data/

# Models
models/*
!models/.gitkeep

# Build
build/
dist/
*.egg-info/
7-4. docs/architecture.md
# Device Architecture

## 책임

NAIT-Device는 사용자와 직접 상호작용하고 센서 데이터를 수집합니다.

## 수행 기능

1. 아바타 UI 표시
2. 고정 질문 표시
3. 터치 답변 수집
4. 카메라 프레임 수집
5. MediaPipe 얼굴 분석
6. 원본 영상 폐기
7. 숫자 형태의 얼굴 지표 생성
8. 중앙 서버로 HTTPS 전송
9. 추가 질문과 결과 표시

## 개인정보 원칙

- 얼굴 원본 영상은 외부로 전송하지 않습니다.
- 얼굴 원본 영상은 기본적으로 저장하지 않습니다.
- 테스트용 이미지에는 개인정보가 포함되지 않도록 합니다.
- `captures/`, `recordings/`, `face_data/`는 Git에 포함하지 않습니다.
8. NAIT-AI
GACHI-AI처럼 별도 FastAPI 서버로 운영하고, 분석 결과 JSON만 반환하며 DB 저장은 BE가 담당하도록 만듭니다. 

8-1. README.md
# NAIT-AI

NAIT 프로젝트의 AI 분석 서버입니다.

Backend와 분리된 FastAPI 애플리케이션으로 운영하며,
표정 지표 해석과 GPT 기반 추가 질문 생성을 담당합니다.

AI 서버는 분석 결과 JSON만 반환하고,
사용자 데이터와 최종 결과 저장은 NAIT-BE가 담당합니다.

## 주요 역할

- MediaPipe 얼굴 지표 해석
- 안전한 표정 패턴 라벨 생성
- GPT 추가 질문 생성
- 구조화된 JSON 응답 생성
- 의료적 진단 표현 방지
- 프롬프트 및 응답 검증

## 처리 흐름

```text
NAIT-BE
  ↓
self_report + facial_features
  ↓
Expression Interpreter
  ↓
GPT Question Generator
  ↓
question + options + avatar_message
  ↓
NAIT-BE
입력 예시
{
  "self_report": {
    "mood_score": 2,
    "sleep_hours": 5,
    "stress_score": 4,
    "activity_level": "low"
  },
  "facial_features": {
    "smile_intensity": 0.12,
    "blink_intensity": 0.06,
    "brow_tension": 0.09,
    "expression_variability": 0.02
  }
}
출력 예시
{
  "expression_interpretation": {
    "primary_pattern": "low_expression_variability",
    "secondary_pattern": "fatigue_signal_like",
    "summary": "표정 변화량이 낮고 눈 깜빡임이 다소 높게 나타났습니다.",
    "confidence": 0.72
  },
  "adaptive_question": {
    "avatar_message": "수면 상태를 한 번 더 확인해볼게요.",
    "question": "오늘 몸이 무겁거나 무기력한 느낌이 있었나요?",
    "options": [
      "네, 꽤 그랬어요",
      "조금 그랬어요",
      "아니요, 괜찮았어요"
    ]
  }
}
기술 스택
Python

FastAPI

Pydantic

OpenAI API

pytest

Ruff

예상 구조
NAIT-AI/
├── app/
│   ├── main.py
│   ├── routers/
│   ├── schemas/
│   ├── services/
│   │   ├── expression_interpreter.py
│   │   └── question_generator.py
│   └── prompts/
├── tests/
├── docs/
├── requirements.txt
├── .env.example
└── README.md
로컬 실행
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8001
Windows PowerShell:

python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8001
안전 원칙
GPT는 의료 진단 모델로 사용하지 않습니다.

사용자의 상태를 단정적으로 표현하지 않습니다.

얼굴 지표는 참고용 패턴으로만 해석합니다.

API Key와 개인정보를 로그에 기록하지 않습니다.

작업 규칙
기본 브랜치: main

작업 브랜치: feat/이슈번호-작업명

main 직접 Push 금지

Pull Request Review 후 Squash Merge


## 8-2. `.env.example`

```env
APP_ENV=development
LOG_LEVEL=INFO

HOST=0.0.0.0
PORT=8001

OPENAI_API_KEY=
OPENAI_MODEL=

OPENAI_ENABLED=false
OPENAI_REQUEST_TIMEOUT=30
OPENAI_ENABLED=false를 기본으로 두면 API Key가 없어도 rule-based 표정 해석 기능을 먼저 개발할 수 있습니다. GACHI-AI도 기본적으로 비용 없는 baseline을 사용하고 명시적으로 활성화했을 때 OpenAI API를 호출하는 방식을 사용합니다. 

8-3. .gitignore
Device의 Python .gitignore와 동일하게 사용하되 다음을 추가합니다.

# Python
__pycache__/
*.py[cod]
.pytest_cache/
.mypy_cache/
.ruff_cache/
.coverage
htmlcov/

# Virtual environments
.venv/
venv/
env/

# Environment
.env
.env.*
!.env.example

# IDE and OS
.vscode/
.idea/
*.iml
.DS_Store
Thumbs.db

# Generated output
logs/
reports/
outputs/
*.log

# Evaluation and temporary data
data/private/
data/raw/
tmp/

# Build
build/
dist/
*.egg-info/
8-4. docs/env.md
# Environment Variables

| 환경변수 | 필수 | 기본값 | 설명 |
|---|---:|---|---|
| `APP_ENV` | 아니요 | `development` | 실행 환경 |
| `LOG_LEVEL` | 아니요 | `INFO` | 로그 레벨 |
| `HOST` | 아니요 | `0.0.0.0` | 서버 호스트 |
| `PORT` | 아니요 | `8001` | 서버 포트 |
| `OPENAI_API_KEY` | 조건부 | 없음 | OpenAI API Key |
| `OPENAI_MODEL` | 조건부 | 없음 | 사용할 OpenAI 모델 |
| `OPENAI_ENABLED` | 아니요 | `false` | OpenAI 호출 활성화 |
| `OPENAI_REQUEST_TIMEOUT` | 아니요 | `30` | 요청 제한 시간 |

실제 값은 `.env`에 작성하고 GitHub에는 `.env.example`만 등록합니다.
8-5. docs/api.md
# AI API

## Health Check

```http
GET /ai/health
표정 해석 및 추가 질문 생성
POST /ai/analysis
Content-Type: application/json
Request
{
  "session_id": "20260711_001",
  "self_report": {
    "mood_score": 2,
    "sleep_hours": 5,
    "stress_score": 4,
    "activity_level": "low"
  },
  "facial_features": {
    "smile_intensity": 0.12,
    "frown_intensity": 0.03,
    "blink_intensity": 0.06,
    "brow_tension": 0.09,
    "eye_engagement": 0.04,
    "expression_variability": 0.02
  }
}
Response
{
  "expression_interpretation": {
    "primary_pattern": "low_expression_variability",
    "secondary_pattern": "fatigue_signal_like",
    "summary": "표정 변화량이 낮게 나타났습니다.",
    "confidence": 0.72
  },
  "adaptive_question": {
    "avatar_message": "한 가지만 더 확인해볼게요.",
    "question": "오늘 몸이 무겁거나 무기력한 느낌이 있었나요?",
    "options": [
      "네, 꽤 그랬어요",
      "조금 그랬어요",
      "아니요, 괜찮았어요"
    ]
  }
}

---

# 9. NAIT-BE

아래 내용은 Spring Boot 기준입니다. 백엔드를 FastAPI나 Node.js로 결정한다면 `.gitignore`와 실행 명령만 바꾸면 됩니다.

## 9-1. `README.md`

```markdown
# NAIT-BE

NAIT 프로젝트의 중앙 Backend 서버입니다.

Device, AI, Database, Mobius 및 Frontend 사이의 중심 역할을 담당합니다.

## 주요 역할

- 사용자 관리
- 기기 등록 및 연결
- 세션 생성 및 관리
- Device 분석 데이터 수신
- NAIT-AI 서버 호출
- 추가 질문 답변 수신
- 최종 컨디션 점수 계산
- Central DB 저장
- Mobius oneM2M 저장
- Frontend 조회 API 제공

## 시스템 경계

```text
NAIT-Device
   ↓
NAIT-BE
   ├─ NAIT-AI
   ├─ Central DB
   ├─ Mobius
   └─ NAIT-FE
AI 서버는 표정 해석과 질문 생성 결과만 반환하며,
DB 저장과 최종 서비스 로직은 Backend에서 담당합니다.

예정 기술 스택
Java

Spring Boot

Spring Data JPA

PostgreSQL 또는 MySQL

Flyway

Mobius oneM2M

Gradle

예상 구조
NAIT-BE/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/nait/
│   │   │       ├── user/
│   │   │       ├── device/
│   │   │       ├── session/
│   │   │       ├── checkin/
│   │   │       ├── score/
│   │   │       └── integration/
│   │   │           ├── ai/
│   │   │           └── mobius/
│   │   └── resources/
│   │       └── db/migration/
│   └── test/
├── docs/
├── .env.example
└── README.md
데이터 처리 흐름
Device가 자가진단 답변과 얼굴 지표를 전송합니다.

Backend가 사용자, 기기 및 세션을 확인합니다.

Backend가 AI 서버에 표정 지표 해석과 질문 생성을 요청합니다.

추가 질문 답변을 수신합니다.

최종 점수를 계산합니다.

결과를 Central DB와 Mobius에 저장합니다.

Device와 Frontend에 결과를 제공합니다.

환경변수
.env.example을 참고해 로컬 환경변수를 설정합니다.

API Key, DB 비밀번호 및 Mobius 인증 정보는 GitHub에 Commit하지 않습니다.

문서
docs/api-response.md: 공통 API 응답 형식

docs/error-code.md: 공통 오류 코드

docs/env.md: 환경변수

docs/database.md: DB 테이블 및 마이그레이션 규칙

작업 규칙
기본 브랜치: main

작업 브랜치: feat/이슈번호-작업명

main 직접 Push 금지

Pull Request Review 후 Squash Merge


GACHI-BE도 `docs/api-response.md`, `docs/error-code.md`, `docs/flyway.md`, `docs/env.md`, `docs/deploy.md`처럼 운영 문서를 분리합니다. NAIT 초기에는 배포 문서보다 API, DB, 환경변수 문서를 먼저 두는 것이 좋습니다. :contentReference[oaicite:7]{index=7}

## 9-2. `.env.example`

```env
APP_ENV=development
SERVER_PORT=8080

DATABASE_URL=jdbc:postgresql://localhost:5432/nait
DATABASE_USERNAME=nait
DATABASE_PASSWORD=

AI_SERVER_URL=http://localhost:8001
AI_SERVER_TIMEOUT=30

MOBIUS_BASE_URL=
MOBIUS_ORIGIN=
MOBIUS_AE_NAME=nait_server

JWT_SECRET=
CORS_ALLOWED_ORIGINS=http://localhost:5173
9-3. .gitignore
# Gradle
.gradle/
build/
out/
target/

# Compiled files
*.class
*.jar
*.war

# Environment and secrets
.env
.env.*
!.env.example

application-local.yml
application-secret.yml
application-prod.yml

# IDE
.idea/
.vscode/
*.iml
.classpath
.project
.settings/

# OS
.DS_Store
Thumbs.db

# Logs
logs/
*.log

# Temporary files
tmp/
9-4. docs/api-response.md
# Common API Response

## 성공 응답

```json
{
  "success": true,
  "data": {},
  "error": null
}
실패 응답
{
  "success": false,
  "data": null,
  "error": {
    "code": "SESSION_NOT_FOUND",
    "message": "세션을 찾을 수 없습니다."
  }
}
원칙
정상 응답과 오류 응답의 구조를 통일합니다.

사용자에게 표시할 메시지와 내부 로그를 분리합니다.

Stack trace나 내부 인증 정보를 API 응답에 포함하지 않습니다.


## 9-5. `docs/error-code.md`

```markdown
# Error Codes

| HTTP | 코드 | 설명 |
|---:|---|---|
| 400 | `INVALID_REQUEST` | 요청 형식이 올바르지 않음 |
| 400 | `INVALID_SELF_REPORT` | 자가진단 응답 범위 오류 |
| 400 | `INVALID_FACIAL_FEATURES` | 얼굴 지표 형식 오류 |
| 401 | `UNAUTHORIZED_DEVICE` | 등록되지 않은 기기 |
| 404 | `USER_NOT_FOUND` | 사용자 없음 |
| 404 | `DEVICE_NOT_FOUND` | 기기 없음 |
| 404 | `SESSION_NOT_FOUND` | 세션 없음 |
| 502 | `AI_SERVER_ERROR` | AI 서버 호출 실패 |
| 502 | `MOBIUS_ERROR` | Mobius 저장 실패 |
| 500 | `INTERNAL_SERVER_ERROR` | 서버 내부 오류 |
9-6. docs/database.md
# Database

## 주요 테이블

### users

- `user_id`
- `kakao_user_id`
- `nickname`
- `created_at`
- `updated_at`

### devices

- `device_id`
- `device_uid`
- `user_id`
- `ae_name`
- `created_at`

### sessions

- `session_id`
- `user_id`
- `device_id`
- `status`
- `created_at`
- `completed_at`

### checkins

- `checkin_id`
- `session_id`
- `mood_score`
- `sleep_hours`
- `stress_score`
- `activity_level`

### facial_features

- `facial_feature_id`
- `session_id`
- `smile_intensity`
- `frown_intensity`
- `blink_intensity`
- `brow_tension`
- `eye_engagement`
- `expression_variability`

### condition_results

- `result_id`
- `session_id`
- `self_report_score`
- `facial_response_score`
- `adaptive_question_score`
- `condition_score`
- `condition_level`
- `recommendation`
- `created_at`

## 마이그레이션 규칙

DB 스키마 변경은 직접 수정하지 않고 마이그레이션 파일로 관리합니다.

예시:

```text
V1__create_users.sql
V2__create_devices.sql
V3__create_sessions.sql

---

# 10. NAIT-FE

React와 TypeScript 기준입니다.

## 10-1. `README.md`

```markdown
# NAIT-FE

NAIT 프로젝트의 사용자 결과 조회용 Frontend입니다.

Backend API를 통해 오늘의 컨디션 결과, 최근 기록과 추천 메시지를 표시합니다.

## 주요 역할

- 사용자 로그인 및 인증 화면
- 오늘의 감정 컨디션 결과 표시
- 컨디션 점수와 상태 표시
- 최근 7일 변화 그래프
- 수면 및 스트레스 기록
- 세션별 기록 조회
- 추천 메시지 표시

## 예정 기술 스택

- React
- TypeScript
- Vite
- React Router
- Backend REST API

## 예상 구조

```text
NAIT-FE/
├── src/
│   ├── pages/
│   ├── components/
│   ├── api/
│   ├── hooks/
│   ├── types/
│   └── assets/
├── public/
├── tests/
├── docs/
├── .env.example
└── README.md
화면 구성
Login
  ↓
Dashboard
  ├─ 오늘의 컨디션
  ├─ 최근 7일 변화
  ├─ 수면 및 스트레스 기록
  └─ 추천 메시지
로컬 실행
프로젝트 생성 후 다음 방식으로 실행합니다.

npm install
npm run dev
환경변수
.env.example을 복사해 .env를 생성합니다.

cp .env.example .env
브라우저에 노출되는 환경변수에는 비밀번호나 API Key를 넣지 않습니다.

작업 규칙
기본 브랜치: main

작업 브랜치: feat/이슈번호-작업명

main 직접 Push 금지

Pull Request Review 후 Squash Merge


## 10-2. `.env.example`

Vite 기준:

```env
VITE_API_BASE_URL=http://localhost:8080
VITE_APP_NAME=NAIT
10-3. .gitignore
# Dependencies
node_modules/

# Build
dist/
build/
.next/
coverage/

# Environment
.env
.env.*
!.env.example

# Logs
npm-debug.log*
yarn-debug.log*
pnpm-debug.log*

# IDE
.vscode/
.idea/
*.iml

# OS
.DS_Store
Thumbs.db

# Cache
.eslintcache
.vite/
10-4. docs/ui.md
# UI Specification

## Dashboard

표시 항목:

- 오늘의 컨디션 점수
- 컨디션 상태
- 최근 7일 변화 그래프
- 수면 기록
- 스트레스 기록
- 추천 메시지

## 상태 예시

- `GOOD`
- `NORMAL`
- `CARE`

상태 이름과 색상은 Backend API 명세와 Frontend 디자인에서 함께 확정합니다.

## 개인정보

- 얼굴 원본 이미지는 Frontend에 전달하지 않습니다.
- 사용자 식별 정보는 필요한 범위에서만 표시합니다.
- 테스트 데이터에 실제 개인정보를 사용하지 않습니다.
11. 아직 만들지 않아도 되는 파일
GACHI에는 아래 파일들도 있지만, NAIT는 아직 실제 코드와 배포 환경이 없으므로 미리 복사하지 마세요. GACHI-AI와 BE는 Docker, CI/CD, 배포 구조가 이미 있어서 해당 파일들을 사용하고 있습니다. 

.coderabbit.yaml
Dockerfile
.dockerignore
.github/workflows/ci.yml
.github/workflows/deploy.yml
dependabot.yml
SECURITY.md
docs/deploy.md
추가 시점은 다음과 같습니다.

파일	추가 시점
.coderabbit.yaml	첫 실제 PR을 만든 뒤
GitHub Actions	실행 가능한 코드와 테스트가 생긴 뒤
Dockerfile	각 서버의 로컬 실행이 안정화된 뒤
docs/deploy.md	배포 서버와 방식이 정해진 뒤
Dependabot	패키지 파일이 확정된 뒤
CD Workflow	배포 환경과 Secret이 정해진 뒤
12. 지금 GitHub에서 만드는 순서
현재 NAIT-Device 화면에서 작업 중이므로 다음 순서가 좋습니다.

1. .github/pull_request_template.md
2. README.md
3. .gitignore
4. .env.example
5. .editorconfig
6. .gitattributes
7. docs/architecture.md
초기 설정 중에는 모두:

Commit directly to the main branch
로 등록해도 됩니다.

커밋 메시지는 파일별로 다음처럼 작성하세요.

chore: Pull Request Template 추가
docs: Device README 작성
chore: Device gitignore 추가
chore: Device 환경변수 예시 추가
chore: EditorConfig 추가
chore: Git attributes 추가
docs: Device 구조 문서 작성
네 레포 파일 작성이 끝난 뒤:

1. Organization에 .github 레포 생성
2. 공통 Issue Template 등록
3. 공통 CONTRIBUTING.md 등록
4. 각 레포 Settings 설정
5. main Ruleset 설정
6. 팀원 초대
7. 승인 1명 필수 설정
순서로 진행하면 됩니다.


14399e2a-f04b-4271-b707-5dcad8252f50.png
프로젝트 초기 세팅 여기선 이렇게 되어 있는데 한번에 할 수 있는 거야? 어떻게 해?

네. 저 화면의 “47 files changed / init: 프로젝트 초기 세팅”은 파일을 하나씩 GitHub 웹에서 만든 게 아니라, 보통 로컬 PC에서 프로젝트 전체 구조를 만든 뒤 한 번에 커밋해서 Push한 것입니다.

NAIT도 이 방식으로 하는 게 훨씬 편합니다.

가장 추천하는 방법
GitHub에서 빈 레포 생성
→ PC로 Clone
→ 폴더와 초기 파일을 한 번에 생성
→ git add .
→ 한 번에 Commit
→ Push
이렇게 하면 다음을 한 커밋으로 올릴 수 있습니다.

.github/
README.md
.gitignore
.env.example
.editorconfig
.gitattributes
docs/
src 또는 app/
requirements.txt / package.json
GitHub 웹에서도 여러 파일을 한 번에 업로드할 수 있지만, .gitattributes의 줄바꿈 처리 등은 웹 업로드에 적용되지 않을 수 있으므로 초기 프로젝트 구성은 Git으로 Push하는 편이 더 안정적입니다. 

방법 1. VS Code와 터미널로 한 번에 올리기
현재 NAIT-Device를 예로 설명하겠습니다.

1. GitHub 레포 주소 복사
NAIT-Device 레포에서:

Code
→ HTTPS
→ 주소 복사
예:

https://github.com/NAIT-Project/NAIT-Device.git
GitHub 공식 절차도 레포의 Code 버튼에서 URL을 복사한 뒤 로컬로 Clone하는 방식입니다. 

2. PC에서 저장할 폴더로 이동
Windows PowerShell 또는 VS Code 터미널을 엽니다.

예를 들어 바탕화면에 작업 폴더를 만들려면:

cd $HOME\Desktop
mkdir NAIT
cd NAIT
이미 NAIT 폴더가 있다면:

cd $HOME\Desktop\NAIT
3. 레포 Clone
git clone https://github.com/NAIT-Project/NAIT-Device.git
cd NAIT-Device
확인:

git status
정상이라면 대략 다음처럼 나옵니다.

On branch main
Your branch is up to date with 'origin/main'.
4. 초기 폴더를 한 번에 생성
PowerShell 기준입니다.

New-Item -ItemType Directory -Force `
  ".github\ISSUE_TEMPLATE", `
  "app\camera", `
  "app\analysis", `
  "app\ui", `
  "app\api", `
  "tests", `
  "docs", `
  "models"
빈 폴더는 Git이 추적하지 않기 때문에 .gitkeep 파일도 만듭니다.

New-Item -ItemType File -Force `
  "app\camera\.gitkeep", `
  "app\analysis\.gitkeep", `
  "app\ui\.gitkeep", `
  "app\api\.gitkeep", `
  "tests\.gitkeep", `
  "models\.gitkeep"
최종 구조:

NAIT-Device/
├── .github/
│   ├── ISSUE_TEMPLATE/
│   │   ├── feature.yml
│   │   ├── bug.yml
│   │   ├── task.yml
│   │   └── config.yml
│   └── pull_request_template.md
├── app/
│   ├── camera/
│   ├── analysis/
│   ├── ui/
│   └── api/
├── tests/
├── docs/
│   └── architecture.md
├── models/
├── README.md
├── .gitignore
├── .env.example
├── .editorconfig
├── .gitattributes
└── requirements.txt
5. 파일을 한 번에 만들기
PowerShell에서:

New-Item -ItemType File -Force `
  ".github\ISSUE_TEMPLATE\feature.yml", `
  ".github\ISSUE_TEMPLATE\bug.yml", `
  ".github\ISSUE_TEMPLATE\task.yml", `
  ".github\ISSUE_TEMPLATE\config.yml", `
  ".github\pull_request_template.md", `
  "README.md", `
  ".gitignore", `
  ".env.example", `
  ".editorconfig", `
  ".gitattributes", `
  "requirements.txt", `
  "docs\architecture.md"
그다음 VS Code로 폴더를 엽니다.

code .
VS Code가 없다면 파일 탐색기에서 직접 열어도 됩니다.

6. 각 파일에 내용 붙여넣기
앞서 작성한 NAIT-Device 초기 파일 내용을 각각 넣습니다.

최소한 다음 파일은 반드시 채우세요.

README.md
.gitignore
.env.example
.github/pull_request_template.md
.github/ISSUE_TEMPLATE/feature.yml
.github/ISSUE_TEMPLATE/bug.yml
.github/ISSUE_TEMPLATE/task.yml
.github/ISSUE_TEMPLATE/config.yml
docs/architecture.md
requirements.txt
아직 버전을 확정하지 않았다면 너무 많이 넣지 마세요.

mediapipe
opencv-python
requests
python-dotenv
라즈베리파이 환경에서는 일부 패키지 설치 방식이 달라질 수 있으므로, 실제 설치 확인 후 버전을 고정하면 됩니다.

초기에는 이렇게 두고 나중에:

mediapipe==버전
opencv-python==버전
형식으로 고정합니다.

7. 변경 파일 확인
git status
다음처럼 여러 파일이 한 번에 표시됩니다.

Untracked files:
  .github/
  .editorconfig
  .env.example
  .gitattributes
  .gitignore
  README.md
  app/
  docs/
  models/
  requirements.txt
  tests/
8. 전체 파일 한 번에 추가
git add .
여기서 .은 현재 폴더 안의 변경 파일 전체를 의미합니다.

추가된 파일 확인:

git status
다음처럼 보이면 정상입니다.

Changes to be committed:
  new file: .github/ISSUE_TEMPLATE/feature.yml
  new file: .github/ISSUE_TEMPLATE/bug.yml
  new file: README.md
  ...
9. 한 번에 Commit
git commit -m "🎉 init: Device 프로젝트 초기 세팅"
이 경우 스크린샷처럼 한 Commit에 여러 파일이 들어갑니다.

다만 팀 컨벤션을 단순하게 하려면 이모지 없이 다음도 좋습니다.

git commit -m "chore: Device 프로젝트 초기 세팅"
둘 중 하나로 통일하세요.

제가 권장하는 형식은:

chore: Device 프로젝트 초기 세팅
레포별로:

chore: Device 프로젝트 초기 세팅
chore: AI 프로젝트 초기 세팅
chore: Backend 프로젝트 초기 세팅
chore: Frontend 프로젝트 초기 세팅
10. GitHub로 Push
git push origin main
GitHub는 로컬 Commit을 원격 저장소에 반영할 때 git push를 사용합니다. 

이제 GitHub에 들어가면 스크린샷처럼:

chore: Device 프로젝트 초기 세팅
여러 files changed
형태로 표시됩니다.

GitHub 웹에서 한 번에 올리는 방법도 있음
로컬 Git이 너무 부담스럽다면 다음 방식도 가능합니다.

Repository
→ Add file
→ Upload files
그다음 PC에서 만든 프로젝트 폴더 안의 파일과 폴더를 드래그합니다.

GitHub는 웹에서 여러 파일을 선택하거나 드래그하여 한 번에 업로드할 수 있습니다. 

커밋 메시지:

chore: Device 프로젝트 초기 세팅
그리고:

Commit directly to the main branch
를 선택합니다.

웹 업로드의 단점
폴더 구조를 먼저 PC에서 만들어야 함

빈 폴더는 올라가지 않음

.gitattributes 처리 결과가 로컬 Git과 다를 수 있음

일부 숨김 파일 선택이 불편함

수정할 때 매번 웹으로 들어가야 함

그래서 처음만이라도 Clone → 파일 생성 → Commit → Push 방식을 익혀두는 것이 좋습니다.

GitHub 웹의 github.dev 편집기를 사용하는 방법
레포 화면에서 키보드의 마침표를 누르면:

.
브라우저 안에서 VS Code와 비슷한 github.dev 편집기가 열립니다.

또는 URL을:

github.com
에서:

github.dev
로 바꿉니다.

예:

https://github.dev/NAIT-Project/NAIT-Device
여기서는 여러 폴더와 파일을 만들고, 왼쪽 Source Control에서 한 번에 Commit할 수 있습니다. GitHub 공식 문서도 일반 웹 편집기 외에 github.dev 편집기를 사용할 수 있다고 안내합니다. 

다만 실제 Python 실행이나 패키지 설치는 할 수 없으므로, 프로젝트 개발은 나중에 로컬 환경에서 해야 합니다.

가장 쉬운 선택 비교
방법	난이도	여러 파일 한 번에	추천도
GitHub에서 파일 하나씩 생성	낮음	어려움	낮음
GitHub Upload files	낮음	가능	보통
github.dev 편집기	보통	가능	보통
로컬 Clone + VS Code + Git	보통	가능	가장 추천
GitHub Desktop	낮음~보통	가능	Git 명령어가 부담될 때 추천
네 레포를 빠르게 만드는 방법
한 레포를 완성한 뒤 모든 파일을 무작정 복사하면 안 됩니다. 공통 파일만 복사하고, 기술별 파일을 바꿔야 합니다.

공통 복사 가능
.github/
.editorconfig
.gitattributes
레포별 수정 필요
README.md
.gitignore
.env.example
requirements.txt
package.json
docs/
추천 작업 순서
1. NAIT-Device를 로컬에서 초기 구성
2. Commit 및 Push
3. NAIT-AI Clone
4. 공통 .github, .editorconfig, .gitattributes 복사
5. AI 전용 README, .env.example, .gitignore 작성
6. Commit 및 Push
7. BE와 FE도 같은 방식
이번 화면처럼 하나의 Commit으로 올릴 범위
초기 세팅 커밋에는 다음을 함께 넣어도 괜찮습니다.

README
환경변수 예시
gitignore
Issue/PR Template
기본 폴더 구조
formatter 설정
초기 패키지 파일
기본 문서
하지만 다음은 별도 Commit이나 별도 Issue로 분리하는 것이 좋습니다.

MediaPipe 실제 구현
GPT API 연결
DB 스키마 구현
화면 컴포넌트 구현
GitHub Actions
Docker
CodeRabbit 설정
즉 초기 커밋은:

프로젝트가 개발을 시작할 수 있는 골격
까지만 넣으세요.

지금 NAIT-Device에서 실행할 명령 요약
git clone https://github.com/NAIT-Project/NAIT-Device.git
cd NAIT-Device

New-Item -ItemType Directory -Force `
  ".github\ISSUE_TEMPLATE", `
  "app\camera", `
  "app\analysis", `
  "app\ui", `
  "app\api", `
  "tests", `
  "docs", `
  "models"

New-Item -ItemType File -Force `
  ".github\ISSUE_TEMPLATE\feature.yml", `
  ".github\ISSUE_TEMPLATE\bug.yml", `
  ".github\ISSUE_TEMPLATE\task.yml", `
  ".github\ISSUE_TEMPLATE\config.yml", `
  ".github\pull_request_template.md", `
  "README.md", `
  ".gitignore", `
  ".env.example", `
  ".editorconfig", `
  ".gitattributes", `
  "requirements.txt", `
  "docs\architecture.md", `
  "app\camera\.gitkeep", `
  "app\analysis\.gitkeep", `
  "app\ui\.gitkeep", `
  "app\api\.gitkeep", `
  "tests\.gitkeep", `
  "models\.gitkeep"

code .
파일 내용을 작성한 다음:

git add .
git commit -m "chore: Device 프로젝트 초기 세팅"
git push origin main
이렇게 하면 스크린샷과 같은 “여러 파일을 한 번에 추가한 초기 세팅 커밋”이 만들어집니다.


한 폴더에 레포 4개를 다 클론 해도 돼? organization은 안해도 돼? 

맞아요. 기본적인 사용법은 README.md에 넣는 게 좋습니다.
다만 모든 내용을 README에 길게 다 넣기보다는 이렇게 나누는 게 깔끔합니다.

README.md
→ 이 레포가 무엇인지, 실행 방법, 작업 흐름 요약

CONTRIBUTING.md
→ 브랜치, 커밋, 이슈, PR 규칙 상세 설명

.github/ISSUE_TEMPLATE
→ 이슈 작성 양식

.github/pull_request_template.md
→ PR 작성 양식
즉, README에는 “처음 온 팀원이 바로 보고 따라할 수 있는 요약”을 넣고, 자세한 규칙은 CONTRIBUTING.md에 넣으면 됩니다.

현재 NAIT 구조도 Device, AI, BE, FE가 역할이 나뉘어 있으므로 각 레포 README에는 해당 레포 역할과 실행법을 쓰고, 공통 협업 방식은 전 레포에서 동일하게 유지하는 게 좋습니다. 설계 문서에서도 라즈베리파이, 중앙 서버, GPT/AI, DB·Mobius, 웹페이지가 분리된 구조로 정리되어 있습니다. 


README.md에 넣으면 좋은 구성
각 레포 README는 대략 이렇게 가면 됩니다.

# NAIT-Device

## 1. 소개

## 2. 주요 기능

## 3. 프로젝트 구조

## 4. 실행 방법

## 5. 환경변수 설정

## 6. 협업 방식

## 7. Issue 작성 방법

## 8. Branch 작성 방법

## 9. Commit 작성 방법

## 10. Pull Request 작성 방법
다만 6~10번은 모든 레포에 똑같이 넣으면 중복이 생기니까, README에는 요약만 넣고 상세는 CONTRIBUTING.md로 연결하는 방식이 좋습니다.

README에 바로 붙여넣을 협업 방식 섹션
아래 내용을 각 레포 README 하단에 추가하면 됩니다.

## 협업 방식

NAIT 프로젝트는 Issue 기반으로 작업을 관리합니다.

모든 기능 개발, 버그 수정, 문서 수정은 아래 순서로 진행합니다.

```text
Issue 생성
→ 작업 Branch 생성
→ 코드 작성
→ Commit
→ Push
→ Pull Request 생성
→ Review
→ Squash Merge
main 브랜치에는 직접 Push하지 않습니다.
초기 프로젝트 세팅 이후의 모든 작업은 Pull Request를 통해 병합합니다.

Issue 작성 방법
작업을 시작하기 전에 먼저 Issue를 생성합니다.

Issue 제목은 다음 형식을 사용합니다.

[FEAT] 새로운 기능
[FIX] 버그 수정
[DOCS] 문서 수정
[CHORE] 설정 및 기타 작업
[REFACTOR] 코드 구조 개선
[TEST] 테스트 코드
[CI] CI/CD 설정
예시:

[FEAT] MediaPipe 얼굴 지표 추출 기능 구현
[FIX] 카메라 연결 실패 시 예외 처리
[DOCS] Device 실행 방법 문서 작성
[CHORE] 프로젝트 초기 폴더 구조 생성
Issue 본문에는 다음 내용을 작성합니다.

1. 작업 목적
2. 세부 작업 목록
3. 완료 조건
4. 참고 자료 또는 관련 문서
Branch 작성 방법
Branch 이름은 Issue 번호를 포함해 작성합니다.

타입/이슈번호-작업명
예시:

feat/12-face-analyzer
fix/18-camera-reconnect
docs/5-readme-update
chore/3-project-setup
refactor/21-score-engine
Branch 타입은 다음 기준을 따릅니다.

타입	의미
feat	새로운 기능
fix	버그 수정
docs	문서 수정
chore	설정, 패키지, 폴더 구조
refactor	동작 변화 없는 코드 개선
test	테스트 코드
ci	GitHub Actions 등 CI 설정
Commit 작성 방법
Commit 메시지는 다음 형식을 사용합니다.

type: 작업 내용 (#이슈번호)
예시:

feat: MediaPipe 얼굴 지표 추출 구현 (#12)
fix: 카메라 연결 실패 예외 처리 (#18)
docs: README 실행 방법 추가 (#5)
chore: 프로젝트 초기 구조 생성 (#3)
Commit 타입은 Branch 타입과 동일하게 사용합니다.

Pull Request 작성 방법
작업이 끝나면 main 브랜치를 대상으로 Pull Request를 생성합니다.

PR 제목은 작업 내용을 명확하게 작성합니다.

예시:

[FEAT] MediaPipe 얼굴 지표 추출 기능 구현
[FIX] 카메라 재연결 오류 수정
[DOCS] Device README 실행 방법 추가
PR 본문에는 다음 내용을 포함합니다.

1. 관련 Issue
2. 작업 내용
3. 테스트 결과
4. 리뷰어가 확인해야 할 부분
5. 실행 화면 또는 로그
관련 Issue는 다음처럼 연결합니다.

Closes #12
이렇게 작성하면 PR이 Merge될 때 해당 Issue가 자동으로 닫힙니다.

Merge 규칙
NAIT 프로젝트는 Squash and merge를 기본으로 사용합니다.

작업 Branch의 여러 Commit은 main 브랜치에 하나의 Commit으로 정리해 병합합니다.

Merge 후 작업 Branch는 삭제합니다.


---

# CONTRIBUTING.md에는 더 자세히 넣기

README에 위 내용을 넣어도 되지만, 레포가 4개라서 중복이 많아집니다. 그래서 추천은 이겁니다.

## 각 레포 README에는 짧게

```markdown
## 협업 규칙

NAIT 프로젝트는 Issue 기반으로 작업합니다.

```text
Issue 생성
→ Branch 생성
→ Commit
→ Pull Request
→ Review
→ Squash Merge
자세한 규칙은 아래 문서를 확인합니다.

Contribution Guide


## Organization `.github/CONTRIBUTING.md`에는 자세히

브랜치 규칙, Commit 규칙, PR 규칙, 보안 규칙을 전부 넣습니다.

이렇게 하면 나중에 규칙을 바꿀 때 **레포 4개 README를 다 수정하지 않아도 되고**, `.github/CONTRIBUTING.md` 하나만 수정하면 됩니다.

---

# 각 레포 README에 넣을 추천 형태

예를 들어 `NAIT-Device README.md`는 이렇게 구성하면 좋아요.

```markdown
# NAIT-Device

NAIT 프로젝트의 Raspberry Pi 디바이스 애플리케이션입니다.

라즈베리파이에서 아바타 UI, 터치 입력, 카메라 입력 및 MediaPipe 얼굴 분석을 수행합니다.

## 주요 역할

- Raspberry Pi 카메라 입력
- MediaPipe Face Landmarker 실행
- 얼굴 landmark 및 blendshape 추출
- 표정 지표 생성
- 고정 자가진단 질문 표시
- 터치 답변 수집
- 중앙 서버 API 호출
- 추가 질문 및 최종 결과 표시

## 프로젝트 구조

```text
NAIT-Device/
├── app/
│   ├── camera/
│   ├── analysis/
│   ├── ui/
│   └── api/
├── tests/
├── docs/
├── models/
├── requirements.txt
├── .env.example
└── README.md
실행 방법
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python -m app.main
Windows PowerShell:

python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
python -m app.main
환경변수 설정
.env.example을 복사해 .env를 생성합니다.

cp .env.example .env
실제 API Key, 비밀번호, 인증 정보는 GitHub에 Commit하지 않습니다.

협업 방식
NAIT 프로젝트는 Issue 기반으로 작업합니다.

Issue 생성
→ 작업 Branch 생성
→ Commit
→ Push
→ Pull Request
→ Review
→ Squash Merge
main 브랜치에는 직접 Push하지 않습니다.

Branch 규칙
타입/이슈번호-작업명
예시:

feat/12-face-analyzer
fix/18-camera-reconnect
docs/5-readme-update
chore/3-project-setup
Commit 규칙
type: 작업 내용 (#이슈번호)
예시:

feat: MediaPipe 얼굴 지표 추출 구현 (#12)
fix: 카메라 재연결 처리 수정 (#18)
docs: README 실행 방법 추가 (#5)
Pull Request 규칙
PR은 main 브랜치를 대상으로 생성합니다.

PR 본문에는 다음 내용을 포함합니다.

관련 Issue

작업 내용

테스트 결과

리뷰 요청 사항

실행 화면 또는 로그

관련 Issue는 다음처럼 연결합니다.

Closes #12
문서
docs/architecture.md: Device 구조 설명

.env.example: 환경변수 예시

.github/ISSUE_TEMPLATE: Issue 양식

.github/pull_request_template.md: PR 양식


---

# 결론

네, **README.md에 기본 사용법을 추가하는 게 맞습니다.**

다만 추천 구조는 이겁니다.

```text
README.md
→ 이 레포를 처음 보는 사람이 바로 이해할 수 있는 요약

CONTRIBUTING.md
→ 팀 전체 협업 규칙 상세 설명

Issue Template
→ 이슈 작성 양식

PR Template
→ PR 작성 양식
지금은 각 레포 README에 협업 방식 요약을 넣고, Organization .github 레포에 CONTRIBUTING.md를 만들어서 자세한 규칙을 관리하는 방식이 가장 좋습니다.


정리해서 만들어줘

좋아요. 아래처럼 **“각 레포 README에는 요약”, “Organization .github 레포에는 상세 규칙”**으로 정리해서 만들면 됩니다.

현재 NAIT 구조는 Device, AI, BE, FE가 역할별로 나뉘어 있고, 라즈베리파이는 데이터 수집과 MediaPipe 분석, 중앙 서버는 저장·점수 계산·Mobius 연동, AI는 GPT 질문 생성, FE는 결과 조회를 담당하는 구조입니다. 


1. 최종 파일 배치
각 레포에 넣을 파일
NAIT-Device/
├── README.md
├── .gitignore
├── .env.example
└── .github/
    └── pull_request_template.md
NAIT-AI/
├── README.md
├── .gitignore
├── .env.example
└── .github/
    └── pull_request_template.md
NAIT-BE/
├── README.md
├── .gitignore
├── .env.example
└── .github/
    └── pull_request_template.md
NAIT-FE/
├── README.md
├── .gitignore
├── .env.example
└── .github/
    └── pull_request_template.md
Organization 공통 .github 레포에 넣을 파일
.github/
├── profile/
│   └── README.md
├── CONTRIBUTING.md
└── ISSUE_TEMPLATE/
    ├── feature.yml
    ├── bug.yml
    ├── task.yml
    └── config.yml
2. 각 레포 README.md에 넣을 공통 협업 섹션
각 레포의 README.md 하단에 아래 내용을 그대로 넣으면 됩니다.

## 협업 방식

NAIT 프로젝트는 Issue 기반으로 작업합니다.

모든 기능 개발, 버그 수정, 문서 수정, 설정 변경은 아래 순서로 진행합니다.

```text
Issue 생성
→ 작업 Branch 생성
→ 코드 작성
→ Commit
→ Push
→ Pull Request 생성
→ Review
→ Squash Merge
초기 프로젝트 세팅 이후에는 main 브랜치에 직접 Push하지 않습니다.

Issue 작성 방법
작업을 시작하기 전에 먼저 Issue를 생성합니다.

Issue 제목은 다음 형식을 사용합니다.

[FEAT] 새로운 기능
[FIX] 버그 수정
[DOCS] 문서 수정
[CHORE] 설정 및 기타 작업
[REFACTOR] 코드 구조 개선
[TEST] 테스트 코드
[CI] CI/CD 설정
예시:

[FEAT] MediaPipe 얼굴 지표 추출 기능 구현
[FIX] 카메라 연결 실패 시 예외 처리
[DOCS] Device 실행 방법 문서 작성
[CHORE] 프로젝트 초기 폴더 구조 생성
Issue 본문에는 다음 내용을 작성합니다.

1. 작업 목적
2. 세부 작업 목록
3. 완료 조건
4. 참고 자료 또는 관련 문서
Branch 작성 방법
Branch 이름은 Issue 번호를 포함해 작성합니다.

타입/이슈번호-작업명
예시:

feat/12-face-analyzer
fix/18-camera-reconnect
docs/5-readme-update
chore/3-project-setup
refactor/21-score-engine
Branch 타입은 다음 기준을 따릅니다.

타입	의미
feat	새로운 기능
fix	버그 수정
docs	문서 수정
chore	설정, 패키지, 폴더 구조
refactor	동작 변화 없는 코드 개선
test	테스트 코드
ci	GitHub Actions 등 CI 설정
Commit 작성 방법
Commit 메시지는 다음 형식을 사용합니다.

type: 작업 내용 (#이슈번호)
예시:

feat: MediaPipe 얼굴 지표 추출 구현 (#12)
fix: 카메라 연결 실패 예외 처리 (#18)
docs: README 실행 방법 추가 (#5)
chore: 프로젝트 초기 구조 생성 (#3)
Commit 타입은 Branch 타입과 동일하게 사용합니다.

Pull Request 작성 방법
작업이 끝나면 main 브랜치를 대상으로 Pull Request를 생성합니다.

PR 제목은 작업 내용을 명확하게 작성합니다.

예시:

[FEAT] MediaPipe 얼굴 지표 추출 기능 구현
[FIX] 카메라 재연결 오류 수정
[DOCS] Device README 실행 방법 추가
PR 본문에는 다음 내용을 포함합니다.

1. 관련 Issue
2. 작업 내용
3. 테스트 결과
4. 리뷰어가 확인해야 할 부분
5. 실행 화면 또는 로그
관련 Issue는 다음처럼 연결합니다.

Closes #12
이렇게 작성하면 PR이 Merge될 때 해당 Issue가 자동으로 닫힙니다.

Merge 규칙
NAIT 프로젝트는 Squash and merge를 기본으로 사용합니다.

작업 Branch의 여러 Commit은 main 브랜치에 하나의 Commit으로 정리해 병합합니다.

Merge 후 작업 Branch는 삭제합니다.


---

# 3. 각 레포 README.md 기본 틀

각 레포 README는 아래 틀로 만들면 됩니다.  
`NAIT-Device`, `NAIT-AI`, `NAIT-BE`, `NAIT-FE`에 맞게 일부만 바꿔서 사용하세요.

```markdown
# NAIT-Device

NAIT 프로젝트의 Raspberry Pi 디바이스 애플리케이션입니다.

라즈베리파이에서 아바타 UI, 터치 입력, 카메라 입력 및 MediaPipe 얼굴 분석을 수행합니다.

## 주요 역할

- Raspberry Pi 카메라 입력
- MediaPipe Face Landmarker 실행
- 얼굴 landmark 및 blendshape 추출
- 표정 지표 생성
- 고정 자가진단 질문 표시
- 터치 답변 수집
- 중앙 서버 API 호출
- 추가 질문 및 최종 결과 표시

## 프로젝트 구조

```text
NAIT-Device/
├── app/
│   ├── camera/
│   ├── analysis/
│   ├── ui/
│   └── api/
├── tests/
├── docs/
├── models/
├── requirements.txt
├── .env.example
└── README.md
실행 방법
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python -m app.main
Windows PowerShell:

python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
python -m app.main
환경변수 설정
.env.example을 복사해 .env를 생성합니다.

cp .env.example .env
실제 API Key, 비밀번호, 인증 정보는 GitHub에 Commit하지 않습니다.


그리고 그 아래에 **2번의 공통 협업 섹션**을 붙이면 됩니다.

---

# 4. 레포별 README 소개 문구

## `NAIT-Device`

```markdown
# NAIT-Device

NAIT 프로젝트의 Raspberry Pi 디바이스 애플리케이션입니다.

라즈베리파이에서 아바타 UI, 터치 입력, 카메라 입력 및 MediaPipe 얼굴 분석을 수행합니다.

원본 얼굴 영상은 외부 서버로 전송하지 않고, 숫자 형태의 표정 지표만 중앙 서버로 전송합니다.
NAIT-AI
# NAIT-AI

NAIT 프로젝트의 AI 분석 서버입니다.

Backend에서 전달받은 자가진단 답변과 얼굴 지표를 바탕으로 표정 패턴을 해석하고,
GPT 기반 맞춤형 추가 질문을 생성합니다.

AI 서버는 분석 결과 JSON만 반환하며, DB 저장은 Backend에서 담당합니다.
NAIT-BE
# NAIT-BE

NAIT 프로젝트의 중앙 Backend 서버입니다.

Device, AI, Central DB, Mobius, Frontend 사이의 중심 역할을 담당합니다.

사용자와 기기 관리, 세션 생성, 분석 데이터 수신, AI 서버 호출, 최종 점수 계산,
DB 저장, Mobius 저장, Frontend 조회 API 제공을 담당합니다.
NAIT-FE
# NAIT-FE

NAIT 프로젝트의 사용자 결과 조회용 Frontend입니다.

Backend API를 통해 오늘의 감정 컨디션 결과, 최근 7일 변화 그래프,
수면 및 스트레스 기록, 추천 메시지를 표시합니다.
5. Organization .github/CONTRIBUTING.md
이 파일은 공통 협업 규칙 상세 문서입니다.

경로:

.github/CONTRIBUTING.md
내용:

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
초기 프로젝트 세팅 이후에는 main 브랜치에 직접 Push하지 않습니다.

2. Repository 구성
Repository	역할
NAIT-Device	Raspberry Pi, 카메라, MediaPipe, 터치형 아바타 UI
NAIT-AI	표정 해석, GPT 추가 질문 생성
NAIT-BE	사용자·기기·세션 관리, DB, 점수 계산, Mobius 연동
NAIT-FE	사용자 결과와 최근 기록을 보여주는 웹페이지
3. Branch 규칙
Branch 이름은 다음 형식을 따릅니다.

타입/이슈번호-작업명
예시:

feat/12-face-analyzer
fix/18-camera-reconnect
docs/5-api-spec
chore/3-project-setup
refactor/21-score-engine
test/25-expression-interpreter
ci/30-python-workflow
Branch 타입
타입	설명
feat	새로운 기능
fix	버그 수정
docs	문서 추가 및 수정
chore	설정, 패키지, 프로젝트 구성
refactor	동작 변화 없는 코드 구조 개선
test	테스트 코드
ci	GitHub Actions 및 CI 설정
4. Commit 규칙
Commit 메시지는 다음 형식을 사용합니다.

type: 작업 내용 (#이슈번호)
예시:

feat: MediaPipe 얼굴 지표 추출 구현 (#12)
fix: 카메라 재연결 처리 수정 (#18)
docs: Device와 BE 사이 API 명세 작성 (#5)
chore: Python 프로젝트 초기 구조 구성 (#3)
Commit 타입
타입	설명
feat	기능 추가
fix	버그 수정
docs	문서 수정
refactor	코드 구조 개선
test	테스트 추가 및 수정
style	포맷, 들여쓰기 등
chore	설정, 의존성, 기타 작업
ci	CI/CD 설정
5. Issue 작성 규칙
작업 시작 전 Issue를 생성합니다.

Issue 제목은 다음 형식을 사용합니다.

[FEAT] 기능 이름
[FIX] 버그 이름
[DOCS] 문서 작업 이름
[CHORE] 설정 작업 이름
[REFACTOR] 리팩토링 대상
[TEST] 테스트 대상
[CI] CI 작업 이름
좋은 예시:

[FEAT] MediaPipe 얼굴 지표 추출 기능 구현
[FIX] 카메라 연결 실패 시 예외 처리
[DOCS] Device 실행 방법 문서 작성
[CHORE] 프로젝트 초기 폴더 구조 생성
좋지 않은 예시:

작업
수정
백엔드 완성
AI 만들기
Issue 하나는 가능한 한 Pull Request 하나로 끝낼 수 있는 크기로 작성합니다.

6. Pull Request 규칙
작업이 끝나면 main 브랜치를 대상으로 Pull Request를 생성합니다.

PR 제목은 다음처럼 작성합니다.

[FEAT] MediaPipe 얼굴 지표 추출 기능 구현
[FIX] 카메라 재연결 오류 수정
[DOCS] README 실행 방법 추가
PR 본문에는 다음을 포함합니다.

관련 Issue

작업 내용

변경 이유

테스트 결과

리뷰 요청 사항

실행 화면 또는 로그

관련 Issue는 다음처럼 연결합니다.

Closes #12
7. Review 규칙
리뷰어는 다음 항목을 확인합니다.

요구사항을 만족하는가

불필요하게 큰 변경이 포함되지 않았는가

실행 방법이 문서화되어 있는가

API Key, 비밀번호, 개인정보가 포함되어 있지 않은가

기존 기능을 깨뜨리지 않는가

에러 상황에 대한 처리가 있는가

리뷰 의견이 남겨진 경우 작성자는 수정 후 다시 확인 요청을 합니다.

8. Merge 규칙
NAIT 프로젝트는 Squash and merge를 기본으로 사용합니다.

작업 Branch의 여러 Commit은 main 브랜치에 하나의 Commit으로 정리해 병합합니다.

Merge 후 작업 Branch는 삭제합니다.

9. 보안 및 개인정보 규칙
다음 항목은 GitHub에 Commit하지 않습니다.

.env

OpenAI API Key

DB 계정과 비밀번호

JWT Secret

Mobius 인증 정보

개인키와 인증서

사용자의 얼굴 원본 사진 및 영상

사용자 개인정보가 포함된 실제 데이터

Repository에는 실제 환경변수 대신 .env.example만 등록합니다.

10. 작업 전 확인 명령어
작업 전에는 항상 최신 main을 가져옵니다.

git switch main
git pull origin main
새 작업 Branch를 생성합니다.

git switch -c feat/12-face-analyzer
작업 후 변경 사항을 확인합니다.

git status
Commit 후 Push합니다.

git add .
git commit -m "feat: MediaPipe 얼굴 지표 추출 구현 (#12)"
git push -u origin feat/12-face-analyzer

---

# 6. `.github/pull_request_template.md`

각 레포에 넣어도 되고, Organization `.github` 레포에 공통으로 넣어도 됩니다.

경로:

```text
.github/pull_request_template.md
내용:

## 관련 Issue

- Closes #

## 작업 내용

- 
- 
- 

## 변경 이유

이 변경이 필요한 이유를 작성해주세요.

## 주요 변경 파일

| 파일 또는 경로 | 변경 내용 |
|---|---|
|  |  |

## 테스트 결과

- [ ] 직접 실행해 정상 동작을 확인했습니다.
- [ ] 기존 기능에 영향을 주지 않는지 확인했습니다.
- [ ] 필요한 예외 처리를 확인했습니다.
- [ ] 필요한 테스트 코드를 추가하거나 수정했습니다.
- [ ] API Key, 비밀번호, 개인정보가 포함되지 않았습니다.
- [ ] 관련 문서를 수정했습니다.

## 리뷰 요청 사항

리뷰어가 집중해서 확인해야 할 부분을 작성해주세요.

## 실행 화면 또는 로그

UI 변경, API 응답, 테스트 결과가 있다면 첨부해주세요.
7. Issue Template
Organization .github 레포에 넣으면 공통으로 사용할 수 있습니다.

ISSUE_TEMPLATE/feature.yml
name: 기능 개발
description: 새로운 기능 구현 작업을 등록합니다.
title: "[FEAT] "
labels:
  - feature
body:
  - type: dropdown
    id: area
    attributes:
      label: 작업 영역
      description: 이 기능이 속한 영역을 선택해주세요.
      options:
        - Device
        - AI
        - Backend
        - Frontend
        - Database
        - Mobius
        - Common
    validations:
      required: true

  - type: textarea
    id: description
    attributes:
      label: 기능 설명
      description: 구현할 기능과 필요한 이유를 작성해주세요.
      placeholder: |
        MediaPipe blendshape 값을 받아서
        smile_intensity, blink_intensity 등의 표정 지표로 변환합니다.
    validations:
      required: true

  - type: textarea
    id: tasks
    attributes:
      label: 작업 항목
      value: |
        - [ ] 요구사항 확인
        - [ ] 기능 구현
        - [ ] 입력값 및 예외 처리
        - [ ] 테스트
        - [ ] 문서 수정
    validations:
      required: true

  - type: textarea
    id: acceptance
    attributes:
      label: 완료 조건
      description: 어떤 상태가 되면 작업을 완료한 것으로 볼지 작성해주세요.
      placeholder: |
        face_analyzer 실행 시 facial_features 형식의 dict가 반환됩니다.
        얼굴이 없는 경우에도 프로그램이 종료되지 않습니다.
    validations:
      required: true

  - type: textarea
    id: reference
    attributes:
      label: 참고 자료
      description: 관련 문서, API, 디자인, 선행 Issue를 작성해주세요.
ISSUE_TEMPLATE/bug.yml
name: 버그 보고
description: 오류 및 예상과 다른 동작을 등록합니다.
title: "[FIX] "
labels:
  - bug
body:
  - type: dropdown
    id: area
    attributes:
      label: 발생 영역
      options:
        - Device
        - AI
        - Backend
        - Frontend
        - Database
        - Mobius
        - Common
    validations:
      required: true

  - type: textarea
    id: problem
    attributes:
      label: 문제 설명
      description: 발생한 문제를 구체적으로 작성해주세요.
    validations:
      required: true

  - type: textarea
    id: reproduction
    attributes:
      label: 재현 방법
      value: |
        1.
        2.
        3.
    validations:
      required: true

  - type: textarea
    id: expected
    attributes:
      label: 기대한 동작
      description: 정상이라면 어떻게 동작해야 하는지 작성해주세요.
    validations:
      required: true

  - type: textarea
    id: environment
    attributes:
      label: 실행 환경
      value: |
        - OS:
        - Device:
        - Python / Java / Node 버전:
        - Browser:
        - 실행 Branch:
    validations:
      required: true

  - type: textarea
    id: logs
    attributes:
      label: 로그 또는 화면
      description: API Key와 개인정보는 제거하고 첨부해주세요.
ISSUE_TEMPLATE/task.yml
name: 일반 작업
description: 프로젝트 설정, 조사, 문서, 운영 작업을 등록합니다.
title: "[TASK] "
labels:
  - chore
body:
  - type: dropdown
    id: branch_type
    attributes:
      label: Branch 타입
      options:
        - chore
        - docs
        - refactor
        - test
        - ci
    validations:
      required: true

  - type: textarea
    id: summary
    attributes:
      label: 작업 요약
      description: 작업 목적과 필요한 이유를 작성해주세요.
    validations:
      required: true

  - type: textarea
    id: tasks
    attributes:
      label: 작업 항목
      value: |
        - [ ] 작업 내용 1
        - [ ] 작업 내용 2
        - [ ] 확인 및 문서화
    validations:
      required: true

  - type: textarea
    id: acceptance
    attributes:
      label: 완료 조건
      description: 작업 완료를 판단할 수 있는 기준을 작성해주세요.
    validations:
      required: true
ISSUE_TEMPLATE/config.yml
blank_issues_enabled: true
contact_links: []
8. 지금 만들 순서
현재 초기 세팅 단계라면 아래 순서로 진행하세요.

1. 각 레포 README.md 작성
2. 각 레포 .gitignore 작성
3. 각 레포 .env.example 작성
4. 각 레포 .github/pull_request_template.md 작성
5. Organization .github 레포 생성
6. .github/CONTRIBUTING.md 작성
7. .github/ISSUE_TEMPLATE 작성
8. main에 초기 세팅 커밋
9. Settings에서 Squash Merge 설정
10. main Ruleset 설정
11. 팀원 초대
초기 세팅 커밋 메시지는 이렇게 쓰면 됩니다.

chore: 프로젝트 초기 세팅
레포별로 구분하고 싶으면:

chore: Device 프로젝트 초기 세팅
chore: AI 프로젝트 초기 세팅
chore: Backend 프로젝트 초기 세팅
chore: Frontend 프로젝트 초기 세팅


1\. 각 레포 README.md 작성 2. 각 레포 .gitignore 작성 3. 각 레포 .env.example 작성 4. 각 레포 .github/pull\_request\_template.md 작성 5. Organization .github 레포 생성 6. .github/CONTRIBUTING.md 작성 7. .github/ISSUE\_TEMPLATE 작성 8. main에 초기 세팅 커밋 9. Settings에서 Squash Merge 설정 10. main Ruleset 설정 11. 팀원 초대

파일들을 완성해줘

완성해뒀어요. 아래 압축 파일을 받아서 각 레포에 복사하면 됩니다.

NAIT GitHub 초기 세팅 파일 다운로드

압축 안에는 이렇게 들어 있습니다.

NAIT-Device/
NAIT-AI/
NAIT-BE/
NAIT-FE/
organization-github/
COMMANDS.md
각 레포에는 다음 파일들이 포함되어 있습니다.

README.md
.gitignore
.env.example
.editorconfig
.gitattributes
.github/pull_request_template.md
docs/
기본 폴더 구조
organization-github 폴더에는 Organization 공통 .github 레포에 넣을 파일들이 들어 있습니다.

profile/README.md
CONTRIBUTING.md
pull_request_template.md
ISSUE_TEMPLATE/
사용 순서는 간단히 이렇게 하면 됩니다.

cd 원하는_레포_폴더
git status
git add .
git commit -m "chore: Device 프로젝트 초기 세팅"
git push origin main
레포별 커밋 메시지는 이렇게 쓰면 됩니다.

chore: Device 프로젝트 초기 세팅
chore: AI 프로젝트 초기 세팅
chore: Backend 프로젝트 초기 세팅
chore: Frontend 프로젝트 초기 세팅
chore: Organization 공통 템플릿 추가
압축 안의 COMMANDS.md에도 적용 방법을 같이 넣어뒀습니다. 


파일 다 올렸는데

좋아요. 그럼 이제 파일 작업은 끝났고, GitHub 설정 단계로 넘어가면 됩니다.

지금 해야 할 순서는 이거예요.

1. GitHub에서 파일이 제대로 올라갔는지 확인
2. 각 레포 Settings 설정
3. main 브랜치 보호 설정
4. Organization 공통 .github 레포 확인
5. GitHub Project 만들기
6. 팀원 초대
7. 첫 Issue/PR 연습
1. 먼저 각 레포에서 확인할 것
NAIT-Device, NAIT-AI, NAIT-BE, NAIT-FE 각각 들어가서 파일이 이렇게 보이면 정상입니다.

README.md
.gitignore
.env.example
.editorconfig
.gitattributes
.github/
docs/
그리고 README가 GitHub 첫 화면에서 잘 보이는지 확인하세요.

특히 .env.example만 올라가고, 실제 .env는 올라가지 않아야 합니다.

올라가도 됨: .env.example
올라가면 안 됨: .env
2. 각 레포 Settings 설정
각 레포마다 반복해서 설정합니다.

Repository
→ Settings
→ General
Features
아래처럼 설정하세요.

Issues: 켜기
Projects: 켜기
Wikis: 끄기
Discussions: 초기에는 끄기
Pull Requests
아래쪽 Pull Requests에서 다음처럼 설정합니다.

Allow merge commits: 끄기
Allow squash merging: 켜기
Allow rebase merging: 끄기
Always suggest updating pull request branches: 켜기
Automatically delete head branches: 켜기
GitHub에서는 PR이 병합된 뒤 작업 브랜치를 자동 삭제하도록 설정할 수 있습니다. 이건 Settings → General → Pull Requests → Automatically delete head branches에서 켭니다. 

3. main 브랜치 보호 설정
각 레포에서:

Settings
→ Rules
→ Rulesets
→ New ruleset
→ New branch ruleset
기본 설정
Ruleset name: Protect main
Enforcement status: Active
Target branches
Add target
→ Include default branch
기본 브랜치가 main이면 main에 적용됩니다.

Rules에서 켤 것
처음에는 이것만 켜세요.

Restrict deletions
Block force pushes
Require a pull request before merging
Require conversation resolution before merging
GitHub ruleset에는 대상 브랜치에 PR을 거쳐서만 병합하도록 요구하는 규칙이 있습니다. 

아직 켜지 말 것
Require status checks to pass
Require deployments to succeed
Require signed commits
Require merge queue
이건 GitHub Actions나 배포가 준비된 뒤 켜는 게 안전합니다.

팀원이 아직 없으면
Required approvals는 지금 바로 1로 하지 않아도 됩니다.

현재는:

Required approvals: 0
또는 승인 필수 설정은 잠시 꺼두세요.

팀원 초대 후, 한 명 이상이 초대를 수락하면 다시 들어가서:

Required approvals: 1
로 바꾸면 됩니다.

4. Organization .github 레포 확인
Organization에 .github 레포도 올렸다면, 그 레포 안에 다음이 있어야 합니다.

profile/README.md
CONTRIBUTING.md
ISSUE_TEMPLATE/
pull_request_template.md
이게 있으면 Organization 첫 화면 소개와 공통 이슈 템플릿이 작동합니다.

다만 각 레포 안에도 .github/pull_request_template.md를 넣어뒀기 때문에 PR 템플릿은 개별 레포 기준으로도 바로 적용됩니다.

5. GitHub Project 만들기
Organization 화면에서:

Projects
→ New project
→ Board
프로젝트 이름은 예를 들어:

NAIT Development
으로 만드세요.

Status는 이렇게 두면 됩니다.

Backlog
Ready
In Progress
Review
Done
사용 방식:

Backlog      아직 정리 중인 작업
Ready        바로 시작 가능한 작업
In Progress  작업 중
Review       PR 확인 중
Done         Merge 완료
6. 초기 Issue 등록
각 레포에 바로 개발 Issue를 너무 많이 만들 필요는 없습니다. 우선 이런 것부터 등록하세요.

Device
[FEAT] MediaPipe 사진 입력 테스트
[FEAT] face_analyzer.py 구현
[FEAT] facial_features JSON 변환
[FEAT] 중앙 서버 전송 기능 구현
AI
[FEAT] expression_interpreter 구현
[FEAT] GPT 추가 질문 프롬프트 작성
[FEAT] GPT 구조화 출력 검증
Backend
[DOCS] Device-BE API 명세 작성
[FEAT] 세션 생성 API 구현
[FEAT] 분석 데이터 수신 API 구현
[FEAT] AI 서버 호출 구현
[FEAT] Mobius 저장 모듈 구현
Frontend
[DESIGN] 결과 화면 와이어프레임 작성
[FEAT] 오늘의 컨디션 결과 화면 구현
[FEAT] 최근 7일 변화 그래프 구현
7. 팀원 초대
이제 팀원 초대해도 됩니다.

Organization에서:

People
→ Invite member
GitHub 아이디나 이메일로 초대할 수 있습니다. GitHub 공식 문서 기준으로 Organization Owner가 사용자 이름이나 이메일 주소를 사용해 팀원을 초대할 수 있습니다. 

권한은 처음에는 이렇게 추천합니다.

팀장: Owner
팀원: Member
각 레포 권한: Write
레포 권한은 Organization에서 레포별로 세부 역할을 줄 수 있습니다. GitHub는 Organization 레포에 대해 사람이나 팀별로 repository-level role을 지정할 수 있습니다. 

8. 팀원 초대 후 꼭 할 것
팀원이 초대를 수락한 뒤, 각 레포 Ruleset으로 다시 가서:

Required approvals: 1
로 바꾸세요.

그러면 앞으로는 누가 코드를 올려도:

작업 브랜치
→ Pull Request
→ 팀원 1명 승인
→ Merge
흐름이 됩니다.

9. 첫 연습 PR 만들기
바로 기능 개발하지 말고, 문서 수정으로 연습하세요.

예를 들어 NAIT-Device에서 Issue 생성:

[DOCS] README 오타 수정 테스트
Issue 번호가 #1이면 브랜치:

docs/1-readme-test
로컬에서:

git switch main
git pull origin main
git switch -c docs/1-readme-test
README에서 문장 하나 수정한 뒤:

git add README.md
git commit -m "docs: README 오타 수정 테스트 (#1)"
git push -u origin docs/1-readme-test
GitHub에서 PR 생성:

base: main
compare: docs/1-readme-test
본문에:

Closes #1
넣고, 팀원이 승인한 뒤 Squash and merge하면 됩니다.

정리하면, 지금 다음으로 할 일은 Settings → General 설정, main Ruleset 설정, Project 생성, 팀원 초대입니다. 기능 개발은 그 다음에 Issue 기반으로 시작하면 됩니다.


이슈 구분? status?를 뭐뭐를 만들고 설명을 어떻게 쓸까?

맞아요. 여기서 헷갈리기 쉬운 게 Issue 구분이랑 Status는 역할이 다릅니다.

Issue 구분 / Labels = 이 일이 어떤 종류인가?
Status = 지금 진행 상태가 어디인가?
예를 들면:

[FEAT] face_analyzer.py 구현
Type: feature
Area: device
Priority: P1
Status: In Progress
현재 NAIT는 Device, AI, Backend, Frontend, Mobius/DB가 나뉘는 구조이므로 Issue도 이 기준으로 구분하면 좋습니다. 설계 문서에서도 라즈베리파이, 중앙 서버, GPT/AI, DB·Mobius, 웹페이지 역할이 분리되어 있습니다. 


1. GitHub Project Status 추천
GitHub Project의 Status는 이 정도만 만들면 됩니다.

Status	설명
Backlog	해야 할 수도 있는 작업. 아직 바로 시작하지 않음
Ready	요구사항이 정리되어 바로 시작 가능한 작업
In Progress	담당자가 작업 중인 Issue
Review	PR을 올렸거나 팀원 확인이 필요한 상태
Blocked	다른 작업, API, 장비, 팀원 확인 때문에 막힌 상태
Done	작업 완료, PR Merge까지 끝난 상태
처음에는 Blocked까지 포함해서 6개를 추천합니다.

Backlog → Ready → In Progress → Review → Done
                    ↓
                 Blocked
Status 설명을 팀원에게 이렇게 안내하면 됨
## Project Status 기준

| Status | 의미 |
|---|---|
| Backlog | 해야 할 작업 후보입니다. 아직 담당자나 일정이 확정되지 않았습니다. |
| Ready | 작업 내용과 완료 조건이 정리되어 바로 시작할 수 있습니다. |
| In Progress | 담당자가 실제로 작업 중입니다. |
| Review | Pull Request가 올라왔거나 팀원 확인이 필요한 상태입니다. |
| Blocked | 외부 요인 때문에 진행이 막힌 상태입니다. 막힌 이유를 Issue 댓글에 남깁니다. |
| Done | Pull Request가 Merge되었고 작업이 완료되었습니다. |
2. Issue Label 추천
Label은 너무 많이 만들면 오히려 안 쓰게 됩니다.
처음에는 Type / Area / Priority 세 종류만 만들면 충분합니다.

2-1. Type Label
Issue가 어떤 종류의 작업인지 나타냅니다.

Label	설명
feature	새로운 기능 구현
bug	오류 수정
docs	README, API 명세, 문서 수정
chore	설정, 폴더 구조, 패키지, 환경 구성
refactor	기능 변화 없이 코드 구조 개선
test	테스트 코드 추가 또는 수정
ci	GitHub Actions, CI/CD 설정
design	화면 설계, UI/UX, 와이어프레임
처음에는 이 8개면 충분합니다.

2-2. Area Label
작업이 어느 파트에 속하는지 나타냅니다.

Label	설명
device	Raspberry Pi, 카메라, MediaPipe, 터치 UI
ai	표정 해석, GPT 질문 생성, 프롬프트
backend	중앙 서버 API, 세션, 점수 계산
frontend	웹 화면, 대시보드, 그래프
database	DB 테이블, 마이그레이션, 저장 구조
mobius	Mobius oneM2M AE/CNT/CIN 저장
api	레포 간 API 명세, 요청/응답 형식
common	여러 레포에 공통으로 적용되는 작업
2-3. Priority Label
중요도를 나타냅니다.

Label	설명
priority: P0	시연이나 전체 진행을 막는 긴급 작업
priority: P1	이번 주 또는 현재 마일스톤에서 꼭 해야 하는 작업
priority: P2	중요하지만 다른 작업 뒤에 해도 되는 작업
priority: P3	여유가 있을 때 하면 좋은 작업
처음에는 P1, P2, P3만 써도 됩니다.
P0는 진짜 막혔을 때만 쓰세요.

3. Label 이름 최종 추천
GitHub Labels에 아래처럼 만들면 됩니다.

Type
feature
bug
docs
chore
refactor
test
ci
design
Area
device
ai
backend
frontend
database
mobius
api
common
Priority
priority: P0
priority: P1
priority: P2
priority: P3
4. Label 설명 문구
GitHub에서 Label 만들 때 Description에 이렇게 넣으면 됩니다.

Label	Description
feature	새로운 기능 구현 작업
bug	오류 또는 예상과 다른 동작 수정
docs	README, API 명세, 사용법 등 문서 작업
chore	프로젝트 설정, 폴더 구조, 패키지 등 기타 작업
refactor	기능 변화 없이 코드 구조를 개선하는 작업
test	테스트 코드 추가 또는 수정
ci	GitHub Actions, CI/CD, 자동화 설정
design	UI/UX, 화면 설계, 와이어프레임 작업
device	Raspberry Pi, 카메라, MediaPipe, 터치 UI 관련
ai	표정 해석, GPT 질문 생성, 프롬프트 관련
backend	중앙 서버 API, 점수 계산, 세션 관리 관련
frontend	웹 화면, 대시보드, 그래프 관련
database	DB 스키마, 테이블, 마이그레이션 관련
mobius	Mobius oneM2M 저장 및 연동 관련
api	레포 간 요청/응답 형식, API 명세 관련
common	여러 레포에 공통으로 적용되는 작업
priority: P0	전체 진행을 막는 긴급 작업
priority: P1	현재 마일스톤에서 반드시 해야 하는 작업
priority: P2	중요하지만 우선순위가 중간인 작업
priority: P3	여유가 있을 때 진행할 작업
5. Issue 제목 구분 추천
Issue 제목은 Label과 별개로 앞에 태그를 붙이는 게 좋습니다.

[FEAT] MediaPipe 얼굴 지표 추출 기능 구현
[FIX] 카메라 연결 실패 시 예외 처리
[DOCS] Device 실행 방법 문서 작성
[CHORE] 프로젝트 초기 폴더 구조 생성
[REFACTOR] 점수 계산 로직 분리
[TEST] 표정 해석 모듈 테스트 추가
[CI] Python CI Workflow 추가
[DESIGN] 결과 화면 와이어프레임 작성
제목 앞 태그와 Label은 보통 맞춰주면 됩니다.

예:

제목: [FEAT] face_analyzer.py 구현
Label: feature, device, priority: P1
Status: Ready
6. GitHub Project 필드 추천
Project에는 Label 말고 별도 Field를 만들 수 있습니다.
처음에는 아래 정도만 추천합니다.

Field	Type	값
Status	Single select	Backlog, Ready, In Progress, Review, Blocked, Done
Priority	Single select	P0, P1, P2, P3
Area	Single select	Device, AI, Backend, Frontend, Database, Mobius, Common
Target date	Date	마감일
Assignees	기본 제공	담당자
Repository	기본 제공	어느 레포인지
Labels에도 Priority와 Area를 만들고 Project Field에도 Priority와 Area를 만들면 중복처럼 보일 수 있습니다.

그래서 처음에는 이렇게 운영하는 걸 추천합니다.

Labels
→ Type 중심으로 사용
feature, bug, docs, chore, refactor, test, ci, design

Project Fields
→ Status, Priority, Area 관리
즉:

Issue Label: feature
Project Area: Device
Project Priority: P1
Project Status: In Progress
이렇게 하면 깔끔합니다.

7. 실제 Issue 예시
Device Issue
제목: [FEAT] MediaPipe 사진 입력 테스트
Label: feature
Area: Device
Priority: P1
Status: Ready
설명:

## 작업 목적

MediaPipe Face Landmarker를 사용해 사진 1장에서 얼굴 landmark와 blendshape가 출력되는지 확인합니다.

## 작업 항목

- [ ] MediaPipe 설치 방법 확인
- [ ] 테스트 이미지 입력 코드 작성
- [ ] landmark 출력 확인
- [ ] blendshape 출력 확인
- [ ] 실행 방법 README에 추가

## 완료 조건

- 사진 1장을 입력했을 때 landmark와 blendshape 결과가 출력됩니다.
- 얼굴이 없는 이미지 입력 시 프로그램이 종료되지 않고 예외 메시지를 출력합니다.
Backend Issue
제목: [DOCS] Device-BE API 명세 작성
Label: docs
Area: Backend
Priority: P1
Status: Ready
설명:

## 작업 목적

Device가 Backend로 전송할 self_report와 facial_features 요청 형식을 확정합니다.

## 작업 항목

- [ ] 요청 URL 정의
- [ ] Request JSON 작성
- [ ] Response JSON 작성
- [ ] Error Response 작성
- [ ] docs/api.md에 정리

## 완료 조건

- Device와 Backend 개발자가 같은 JSON 형식을 기준으로 개발할 수 있습니다.
AI Issue
제목: [FEAT] expression_interpreter 구현
Label: feature
Area: AI
Priority: P1
Status: Ready
설명:

## 작업 목적

MediaPipe 얼굴 지표를 직접 감정 진단으로 사용하지 않고 안전한 표정 패턴 라벨로 변환합니다.

## 작업 항목

- [ ] 입력 스키마 정의
- [ ] low_expression_variability 규칙 구현
- [ ] fatigue_signal_like 규칙 구현
- [ ] confidence 계산 방식 임시 정의
- [ ] 테스트 케이스 작성

## 완료 조건

- facial_features 입력 시 expression_interpretation JSON이 반환됩니다.
- 의료 진단처럼 보이는 단정 표현을 사용하지 않습니다.
Frontend Issue
제목: [DESIGN] 결과 화면 와이어프레임 작성
Label: design
Area: Frontend
Priority: P2
Status: Ready
설명:

## 작업 목적

사용자가 오늘의 컨디션 결과와 최근 변화를 확인할 수 있는 화면 구조를 설계합니다.

## 작업 항목

- [ ] 오늘의 점수 카드 설계
- [ ] 상태 표시 영역 설계
- [ ] 최근 7일 그래프 영역 설계
- [ ] 추천 메시지 영역 설계

## 완료 조건

- 팀원이 결과 화면 구성을 이해할 수 있는 와이어프레임이 작성됩니다.
8. 최종 추천 세트
처음에는 이렇게만 만들면 됩니다.

Status
Backlog
Ready
In Progress
Review
Blocked
Done
Label
feature
bug
docs
chore
refactor
test
ci
design
Project Field
Priority: P0, P1, P2, P3
Area: Device, AI, Backend, Frontend, Database, Mobius, Common
Target date
이 정도면 팀원이 많지 않아도 충분히 관리됩니다.
너무 많은 Label을 만들기보다, Status는 진행 상태, Label은 작업 종류, Area는 담당 영역, Priority는 중요도로 역할을 나누는 게 제일 깔끔합니다.


팀원한테 배포할 설명 파일 작성해줘

팀원에게 배포할 설명 파일을 Markdown 형식으로 작성해뒀어요.

NAIT_Team_GitHub_Guide.md

내용에는 아래 항목을 포함했습니다.

- 프로젝트 레포 구조
- 처음 참여할 때 해야 할 일
- Issue 작성 방법
- GitHub Project Status 설명
- Label / Area / Priority 기준
- Branch 이름 규칙
- Commit 메시지 규칙
- Pull Request 작성 방법
- Review와 Merge 방식
- 보안 규칙
- 자주 하는 실수
- Repository별 첫 작업 예시
이 파일을 팀 단톡방에 배포하거나, Organization .github 레포의 CONTRIBUTING.md 또는 각 레포 docs/team-guide.md로 올려도 됩니다. 

레포마다 repository details 만들어줘

라이브러리
/
NAIT
/
NAIT_Team_GitHub_Guide.md


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

작업할 때 이것만 기억하면 됩니다.

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
라이브러리
/
NAIT
/
NAIT_Team_GitHub_Guide.md


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

작업할 때 이것만 기억하면 됩니다.

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
